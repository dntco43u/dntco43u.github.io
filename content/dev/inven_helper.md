---
date: 2025-12-10
title: inven-helper
description: inven 출석 봇
tags:
  - python
  - selenium
categories:
  - dev
---

![image](/images/selenium-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/inven_helper)

## container 구성

### .env
```sh
vi /opt/python/.env
```
```ini
...
INVEN_USERNAME=n*******
INVEN_PASSWORD=5***************************************************************
...
```

### inven_helper.sh [^1]
```sh
#!/bin/bash
# inven_helper

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

for ((i=1; i<=2; i++)); do
  docker run \
    -i --rm --name=py-inven-helper --network=dev --user=0:0 \
    --env-file=/opt/python/.env \
    -v /home/dev/workspace/inven_helper/inven_helper.py:/data/inven_helper.py:rw \
    e7hnr8ov/python:3-slim \
    python /data/inven_helper.py > "$log_file"
  cp "$log_file" "$msg_file"
  send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
  rm "$msg_file"
  sleep 5
done
```

### inven_helper.py [^2]
```py
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as ec
from lxml.html import fromstring
import os, sys, time

try:
  executor = str(os.environ.get("WEBDRIVER_URL"))
  options = webdriver.ChromeOptions()
  #options.add_argument('--disable-gpu')
  options.add_argument('--headless')
  options.add_argument('--blink-settings=imagesEnabled=false')
  options.add_argument('--no-sandbox')
  options.add_argument('--disable-dev-shm-usage')
  options.add_argument('--lang=en_US')
  options.add_argument('--mute-audio')
  options.add_argument('user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/107.0.0.0 Safari/537.36 Edg/107.0.1418.35')
  driver = webdriver.Remote(command_executor=executor, options=options)
  driver.set_window_size(1280, 720)

  # https://it.inven.co.kr/
  url = 'https://it.inven.co.kr/'
  driver.switch_to.window(driver.window_handles[0])
  driver.get(url)
  print('driver.get=' + url)
  try:
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="itBody"]/div[1]/section/article/section[1]/div[1]/div/a')))
    driver.find_element(By.XPATH, '//*[@id="itBody"]/div[1]/section/article/section[1]/div[1]/div/a').click()
  except:
    pass
  print('debug:1')

  # https://member.inven.co.kr/user/scorpio/mlogin
  try:
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="loginBtn"]')))
    username = str(os.environ.get("INVEN_USERNAME"))
    password = str(os.environ.get("INVEN_PASSWORD"))
    # print('INVEN_USERNAME=' + username)
    # print('INVEN_PASSWORD=' + password)
    driver.find_element(By.XPATH, '//*[@id="user_id"]').send_keys(username)
    driver.find_element(By.XPATH, '//*[@id="password"]').send_keys(password)
    driver.find_element(By.XPATH, '//*[@id="password"]').send_keys(Keys.ENTER)
  except:
    pass
  print('debug:2')

  # Change Password
  try:
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="btn-extend"]')))
    driver.find_element(By.XPATH, '//*[@id="btn-extend"]').click()
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="btn-ok"]')))
    driver.find_element(By.XPATH, '//*[@id="btn-ok"]').click()
  except:
    pass
  print('debug:3')

  # https://it.inven.co.kr/
  try:
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="itBody"]/div[1]/section/article/section[1]/div[1]/div/form/ul/li[6]/button')))
    driver.find_element(By.XPATH, '//*[@id="comItBanner"]/div[4]/a/span').click()
  except:
    pass
  print('debug:4')
  # Guaranteed tab open time
  time.sleep(10)

  # https://imart.inven.co.kr/attendance/ Check Attendance
  driver.switch_to.window(driver.window_handles[1])
  try:
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="invenAttendCheck"]/div/div[2]/div/div[3]/div[1]/div[4]/a')))
    driver.find_element(By.XPATH, '//*[@id="invenAttendCheck"]/div/div[2]/div/div[3]/div[1]/div[4]/a').click()
  except:
    pass
  print('debug:5')

  try:
    wait = WebDriverWait(driver, 3).until(ec.alert_is_present())
    alert = driver.switch_to.alert
    print('alert.text=' + alert.text)
    alert.accept()
  except:
    pass
  print('debug:6')

  # if time.localtime().tm_wday == 2:
  try:
    driver.find_element(By.XPATH, '//*[@id="invenAttendCheck"]/div/div[2]/div/div[3]/div[1]/div[3]/a').click()
    driver.switch_to.window(driver.window_handles[2])
    wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="imageCollectDiv"]/div/div[2]/div[2]/div[1]/div[19]/div/label/span')))
    vote_text = driver.find_element(By.XPATH, '//*[@id="imageCollectDiv"]/div/div[2]/div[2]/div[2]').text
    if vote_text == '투표에 참여 완료했습니다.':
      print('div.text=' + vote_text)
    else:
      driver.find_element(By.XPATH, '//*[@id="imageCollectDiv"]/div/div[2]/div[2]/div[1]/div[19]/div/label/span').click()
      driver.find_element(By.XPATH, '//*[@id="imageCollectDiv"]/div/div[2]/div[2]/div[2]/div/button').click()
    driver.close()
    driver.switch_to.window(driver.window_handles[1])
  except:
    pass
  driver.close()
  print('debug:7')

  # https://it.inven.co.kr/
  driver.switch_to.window(driver.window_handles[0])
  driver.find_element(By.XPATH, '//*[@id="comItBanner"]/div[5]/a/span').click()
  print('debug:8')

  # https://imart.inven.co.kr/imarble/ Dice
  for i in range(8):
    driver.switch_to.window(driver.window_handles[1])
    try:
      wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="imarbleBoard"]/div[4]/img')))
      driver.find_element(By.XPATH, '//*[@id="imarbleBoard"]/div[4]/img').click()
    except:
      pass
    try:
      wait = WebDriverWait(driver, 3).until(ec.alert_is_present())
      alert = driver.switch_to.alert
      alert_text = alert.text
      print('alert.text=' + alert.text.replace("\n", " "))
      alert.accept()
      if alert_text.startswith('주사위 1개를 \'5제니\'로 구매하실 수 있습니다.'):
        try:
          wait = WebDriverWait(driver, 3).until(ec.alert_is_present())
          alert = driver.switch_to.alert
          alert.accept()
        except:
          pass
      if alert_text == '오늘의 주사위를 모두 사용하셨습니다.':
        driver.close()
        break
    except:
      pass
    time.sleep(10)
    driver.execute_script('window.open("about:blank", "_blank");')
    driver.switch_to.window(driver.window_handles[2])
    url = 'https://imart.inven.co.kr/imarble/'
    driver.get(url)
    print('driver.get=' + url)
    driver.switch_to.window(driver.window_handles[1])
    driver.close()
  print('debug:9')

  # https://it.inven.co.kr/ Advertisement
  driver.switch_to.window(driver.window_handles[0])
  try:
    wait = WebDriverWait(driver, 5).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="topskyAd"]/div/img')))
    xpaths = ['//*[@id="topskyAd"]/div/img', '//*[@id="rightThumbNailAd"]/img']
    for j in xpaths:
      # print("xpaths=" + j)
      driver.find_element(By.XPATH, j).click()
      driver.switch_to.window(driver.window_handles[1])
      driver.close()
      driver.switch_to.window(driver.window_handles[0])
  except:
    pass
  print('debug:10')

  # https://it.inven.co.kr/
  driver.switch_to.window(driver.window_handles[0])
  res = driver.page_source
  parser = fromstring(res)
  article_list = parser.xpath('//*[@id="commu-nav"]/ul/li[6]/ol')
  parsed_articles = article_list[0].xpath('.//li')
  links = []
  for article in parsed_articles:
    parsed_link = article.xpath('.//a[@href]')
    link = parsed_link[0].get('href')
    links.append(link)
  # print("links=" + str(links))
  print('debug:11')

  # https://links.inven.co.kr/ Brand partners
  for i in links:
    # https://zotac.inven.co.kr/ No like button
    if i == 'https://zotac.inven.co.kr/':
      break
    driver.switch_to.window(driver.window_handles[0])
    driver.execute_script('window.open("about:blank", "_blank");')
    driver.switch_to.window(driver.window_handles[1])
    driver.get(i)
    print('driver.get=' + i)
    try:
      wait = WebDriverWait(driver, 3).until(ec.element_to_be_clickable((By.XPATH, '//*[@id="brandSubscribe"]/div[1]/button')))
      for j in range(3):
        driver.find_element(By.XPATH, '//*[@id="brandSubscribe"]/div[1]/button').click()
        if j <= 1:
          time.sleep(5)
    except:
      pass
    driver.close()
  print('debug:12')

  # close
  driver.switch_to.window(driver.window_handles[0])
  driver.close()
  driver.quit()
  print('debug:13')

except Exception as error:
  print(error)
  pass
```

## Troubleshooting
{{% alert color=warning %}}
> 간헐적으로 실행 도중 종료

원인 찾지 못함. 일단 sh에서 2회 실행
{{% /alert %}}

[^1]: https://github.com/dntco43u/s6h7k8rv/blob/main/inven_helper.sh
[^2]: https://github.com/dntco43u/inven_helper/blob/main/inven_helper.py
