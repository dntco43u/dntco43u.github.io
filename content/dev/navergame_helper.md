---
date: 2025-12-10
title: navergame-helper
description:  navergame 출석 봇
tags:
  - python
  - selenium
categories:
  - dev
---

![image](/images/selenium-logo.webp#center)

## GitHub repository
[바로 가기](https://github.com/dntco43u/navergame_helper)

## container 구성

### .env
```sh
vi /opt/python/.env
```
```ini
...
NAVER_USERNAME=u*******@naver.com
NAVER_PASSWORD=r***************
...
```

### navergame_helper.sh [^1]
```sh
#!/bin/bash
# navergame_helper

source /home/dev/.bashrc
source /home/dev/.local/bin/utils.sh
log_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').log
msg_file=/home/dev/.local/log/$(basename "$0" | sed 's/.sh//').tmp

for ((i=1; i<=2; i++)); do
  docker run \
    -i --rm --name=py-navergame-helper --network=dev --user=0:0 \
    --env-file=/opt/python/.env \
    -v /home/dev/workspace/navergame_helper/navergame_helper.py:/data/navergame_helper.py:rw \
    e7hnr8ov/python:3-slim \
    python /data/navergame_helper.py > "$log_file"
  cp "$log_file" "$msg_file"
  send_tel_msg "$TEL_BOT_KEY" "$TEL_CHAT_ID" "$msg_file"
  rm "$msg_file"
  sleep 5
done
```

### navergame_helper.py [^2]
```py
from selenium import webdriver # type: ignore
from selenium.webdriver.common.by import By # type: ignore
from selenium.webdriver.support.ui import WebDriverWait # type: ignore
from selenium.webdriver.support import expected_conditions as ec # type: ignore
from lxml.html import fromstring # type: ignore
import os, sys, time, random

escape_html_table = {
  "&": "&amp;",
  '"': "&quot;",
  "'": "&apos;",
  ">": "&gt;",
  "<": "&lt;",
}

def escape_html(text):
  return "".join(escape_html_table.get(c,c) for c in text)

def get_ticket_page():
  driver.get('https://game.naver.com/ticket')
  time.sleep(3) #동적 element
  btn_collect_xpath = '//*[@id="root"]/div[1]/div[2]/div/button[2]/span' #티켓 모으기
  WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_collect_xpath))).click()
  time.sleep(3) #동적 element

try:
  executor = str(os.environ.get("WEBDRIVER_URL"))
  #print(executor)
  options = webdriver.ChromeOptions()
  options.add_argument('--headless')
  options.add_argument("--disable-blink-features=AutomationControlled")
  options.add_argument('--blink-settings=imagesEnabled=false')
  options.add_argument('--mute-audio')
  options.add_argument('--no-sandbox')
  options.add_argument('--disable-dev-shm-usage')
  options.add_argument('--mute-audio')
  options.add_argument("--start-maximized")
  options.add_argument('--lang=en_US')
  user_agent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36 Edg/126.0.0.0'
  options.add_argument('user-agent=' + user_agent)
  driver = webdriver.Remote(command_executor=executor, options=options)

  #로그인 https://nid.naver.com/nidlogin.login
  try:
    username = str(os.environ.get("NAVER_USERNAME"))
    password = str(os.environ.get("NAVER_PASSWORD"))
    #print(username + '/' + password)
    driver.get('https://nid.naver.com/nidlogin.login')
    btn_login_xpath = '//*[@id="log.login"]/span'
    btn_login = WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_login_xpath)))
    input_id = driver.find_element(By.XPATH, '//*[@id="id"]')
    input_pw = driver.find_element(By.XPATH, '//*[@id="pw"]')
    driver.execute_script('arguments[0].value=arguments[1]', input_id, username)
    time.sleep(random.uniform(1,3)) #봇 감지 회피
    driver.execute_script('arguments[0].value=arguments[1]', input_pw, password)
    time.sleep(random.uniform(1,3)) #봇 감지 회피
    btn_login.click()
    print('[x] 로그인')
  except Exception as e1:
    print('[ ] 로그인')
    pass

  #기기 등록 요청
  #등록하지 않음, 기기 등록을 하게 되면 실행시 불필요한 네이버 알림 추가됨
  #//*[@id="new.save"] //*[@id="new.dontsave"]
  try:
    btn_dont_xpath = '//*[@id="new.dontsave"]'
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_dont_xpath))).click()
    print('[x] 기기 등록 요청')
  except Exception as e1:
    print('[ ] 기기 등록 요청')
    pass

  #네이버게임 롤 라운지 https://game.naver.com/lounge/League_of_Legends/home
  try:
    is_success = False
    driver.get('https://game.naver.com/lounge/League_of_Legends/home')
     #XXX: 봇 감지될 시 http 404
    time.sleep(6)
    res = driver.page_source
    parser = fromstring(res)
    xpaths = ['//*[@id="root"]/div/div/div[3]/div[1]/div/div[2]/div[2]/ul/li[1]/a'
            , '//*[@id="root"]/div/div/div[3]/div[1]/div/div[3]/div[2]/ul/li[1]/a']
    for xpath in xpaths:
      elem = parser.xpath(xpath)
      if(len(elem) > 0):
        link_latest_xpath = xpath #라운지 최신 글
        is_success = True
        break
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, link_latest_xpath))).click()
    print('[x] 일일 퀘스트 최신 글 보기')
    if (is_success == False):
      print('[ ] 일일 퀘스트 최신 글 보기\n' + link_latest_xpath)
  except Exception as e11:
    #print('e: ' + escape_html(str(121)))
    pass

  #일일 퀘스트 버프
  try:
    driver.refresh()
    time.sleep(6)
    #page_load_xpath = '//*[@id="panelCenter"]/div/div/div[3]'
    #WebDriverWait(driver, 6).until(ec.presence_of_element_located((By.XPATH, page_load_xpath)))
    res = driver.page_source
    parser = fromstring(res)
    xpath = ''
    xpaths = ['//*[@id="panelCenter"]/div/div/div[2]/div[2]/div/button[1]/em'
            , '//*[@id="panelCenter"]/div/div/div[2]/div[3]/div/button[1]/em'
            , '//*[@id="panelCenter"]/div/div/div[3]/div[2]/div/button[1]/em'
            , '//*[@id="panelCenter"]/div/div/div[3]/div[3]/div/button[1]/em']
    for xpath in xpaths:
      elem = parser.xpath(xpath)
      #print(str(len(elem)) + xpath)
      if(len(elem) > 0 and '버프' in elem[0].text):
        btn_buff_xpath = xpath.replace('/em', '')
        break
    btn_buff = WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_buff_xpath)))
    btn_buff.click()
    time.sleep(1)
    btn_buff.click()
    print('[x] 일일 퀘스트 버프하기')
  except Exception as e10:
    print('[ ] 일일 퀘스트 버프하기\n' + btn_buff_xpath)
    pass

  #네이버게임 라운지 티켓 https://game.naver.com/ticket
  #button 주소        //*[@id="root"]/div[2]/div[4]/ul + idx + ]/div/button/div[2]
  #disalbed 확인 주소 //*[@id="root"]/div[2]/div[4]/ul + idx + ]/div/button
  try:
    driver.get('https://game.naver.com/ticket')
    time.sleep(3) #동적 element
    res = driver.page_source
    parser = fromstring(res)
    xpath = ''
    xpaths = ['//*[@id="root"]/div[2]/div[3]/ul/li[1]/div/button/div[2]'
            , '//*[@id="root"]/div[2]/div[4]/ul/li[1]/div/button/div[2]']
    for xpath in xpaths:
      elem = parser.xpath(xpath)
      if(len(elem) > 0 and '응모' in elem[0].text):
        prefix_xpath = xpath.replace('/li[1]/div/button/div[2]', '')
        break
  except Exception as e21:
    print('e: 요소 탐색 실패\n' + prefix_xpath)
    #print('e: ' + escape_html(str(e21)))
    pass
  try:
    idx = 1
    suffix_xpath = ']/div/button/div[2]'
    btn_ticket_xpath = prefix_xpath + '/li[' + str(idx) + suffix_xpath
    #print(btn_ticket_xpath)
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_ticket_xpath)))
    res = driver.page_source
    parser = fromstring(res)
    ul = parser.xpath(prefix_xpath)
    lis = ul[0].xpath('.//li')
    xpaths = []
    for li in lis:
      ticket_name = li.xpath('.//div/div[2]/p')[0].text
      #print('idx=' + str(idx))
      #print('name=' + ticket_name)
      if ticket_name.startswith('네이버페이'):
        xpaths.append(prefix_xpath + '/li[' + str(idx) + suffix_xpath)
      idx += 1
    #print(prefix_xpath)
  except Exception as e22:
    #print('e: 요소 탐색 실패\n' + prefix_xpath)
    pass
  try:
    is_success = False
    for xpath in xpaths:
      try:
        btn_challenge_xpath = xpath.replace('/button/div[2]', '/button')
        btn_challenge = driver.find_element(By.XPATH, btn_challenge_xpath)
        if not (btn_challenge.get_attribute('disabled')):
          #print ('active ticket xpath: ' + xpath)
          WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, xpath))).click()
          is_success = True
          print('[x] 일일 퀘스트 응모 도전')
          break
      except Exception as e2:
        print('e :' + xpath)
        pass
    if (is_success == False):
      print('[ ] 일일 퀘스트 응모 도전\n' + xpath)
  except Exception as e10:
    #print('e: 요소 탐색 실패 ' + escape_html(str(e10)))
    pass

  #일일 퀘스트 완료 보상
  #기준 xpath 검색
  try:
    get_ticket_page()
    is_success = False
    res = driver.page_source
    parser = fromstring(res)
    xpaths = ['//*[@id="root"]/div[2]/div[3]/ul/li[1]/div/div[3]/button'
            , '//*[@id="root"]/div[2]/div[4]/ul/li[1]/div/div[3]/button']
    for xpath in xpaths:
      elem = parser.xpath(xpath)
      if(len(elem) > 0 and len(elem[0].text) > 0): #하러가기, 티켓받기, 보상받기, 완료
        is_success = True
        prefix_xpath = xpath.replace('/ul/li[1]/div/div[3]/button', '')
        break
    if (is_success == False):
      print('e: 요소 탐색 실패\n' + prefix_xpath)
  except Exception as e20:
    #print('e: ' + escape_html(str(e20)))
    pass
  try:
    get_ticket_page()
    btn_today_xpath = prefix_xpath + '/ul/li[1]/div/div[3]/button' #오늘의 티켓 받기
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_today_xpath))).click()
    print('[x] 오늘의 티켓 받기')
  except Exception as e3:
    print('[ ] 오늘의 티켓 받기\n' + btn_today_xpath)
    pass
  try:
    get_ticket_page()
    btn_view_xpath = prefix_xpath + '/ul/li[2]/div[1]/div[3]/button' #가입 라운지에서 최신 글 보기
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_view_xpath))).click()
    print('[x] 최신 글 보기 받기')
  except Exception as e4:
    print('[ ] 최신 글 보기 받기\n' + btn_view_xpath)
    pass
  try:
    get_ticket_page()
    btn_buff_xpath = prefix_xpath + '/ul/li[3]/div[1]/div[3]/button' #게임 라운지에서 버프하기
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_buff_xpath))).click()
    print('[x] 라운지에서 버프하기 받기')
  except Exception as e5:
    print('[ ] 리운지에서 버프하기 받기\n' + btn_buff_xpath)
    pass
  try:
    get_ticket_page()
    btn_challenge_xpath = prefix_xpath + '/ul/li[4]/div[1]/div[3]/button' #라운지 티켓으로 응모 도전
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_challenge_xpath))).click()
    print('[x] 응모 도전 받기')
  except Exception as e6:
    print('[ ] 응모 도전 받기\n' + btn_challenge_xpath)
    pass
  try:
    get_ticket_page()
    btn_reward_xpath = prefix_xpath + '/div/div[1]/div[2]/button' #일일 퀘스트 완료 보상
    WebDriverWait(driver, 6).until(ec.element_to_be_clickable((By.XPATH, btn_reward_xpath))).click()
    print('[x] 일일 퀘스트 완료 보상 받기')
  except Exception as e7:
    print('[ ] 일일 퀘스트 완료 보상 받기\n' + btn_reward_xpath)
    pass
except Exception as e:
  print('e: ' + escape_html(str(e)))
finally:
  driver.quit()
```

## Troubleshooting
{{% alert color=warning %}}
> 간헐적으로 실행 도중 종료

원인 찾지 못함. 일단 sh에서 2회 실행
{{% /alert %}}

[^1]: https://github.com/dntco43u/s6h7k8rv/blob/main/navergame_helper.sh
[^2]: https://github.com/dntco43u/navergame_helper/blob/main/navergame_helper.py
