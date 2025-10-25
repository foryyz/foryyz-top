---
title: '一个基于selenium的半自动淘宝爬虫脚本'
date: 2025-09-26T22:00:03+08:00
draft: false
Tags: ['Crawler', 'Script']
---

# TaoBao Crawler

## 测试环境

- Mac OS
- python=3.12
- 浏览器=Edge

## 运行方法

1. 下载自己浏览器对应的selenium webdriver内核，替换**run.py**中的`EDGE_DRIVER_PATH`参数 （不懂问大模型）
2. **执行run.py**，第一次执行需要手动登陆，登陆后在终端输入save保存cookie，这样以后使用程序就能自动登录了
3. 登陆完成后 在浏览器手动搜索商品（请确保商品不止一页数据）
4. 搜索完成后**在终端输入start**和爬取的页数, 程序会自动完成翻页 保存渲染的html文件
5. 爬取html文件后，执行**save2csv.py**文件对数据进行提取 (可以自己更新**save2csv.py**提取想要的字段)

> [!NOTE]
>
> 如想爬取2级网页 如商品评价等数据同理，进入页面等待渲染完成保存html文件进行数据提取就ok

## 代码

**run.py**

```python
# -*- coding: utf-8 -*-
"""
淘宝爬虫辅助工具
功能：
1. 启动 Edge 浏览器并加载淘宝首页。
2. 若 cookie.pkl 存在则自动注入 cookie 自动登录，否则等待人工登录。
3. 登录后输入命令“已登录”保存 cookie。
4. 进入搜索页面后输入命令“保存页面”保存当前渲染后的 HTML。
"""

from selenium import webdriver
from selenium.webdriver.edge.service import Service
from selenium.webdriver.edge.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import pickle
import time
import os
import random

# ========== 配置部分 ==========
EDGE_DRIVER_PATH = "/Users/yyz/Drivers/EdgeDriver_Mac/msedgedriver"
COOKIES_FILE = "cookies.pkl"
SAVE_DIR = "saved_pages"
os.makedirs(SAVE_DIR, exist_ok=True)

# ========== 初始化浏览器 ==========
def init_driver():
    options = Options()
    # 不要用无头模式，方便人工登录
    options.add_argument("--start-maximized")
    options.add_argument("--disable-blink-features=AutomationControlled")
    options.add_argument("--disable-infobars")
    options.add_argument("--no-sandbox")
    options.add_argument("--disable-gpu")
    # 模拟真实浏览器 UA
    options.add_argument(
        "user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) "
        "AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.5993.88 Safari/537.36 Edg/118.0.2088.61"
    )
    service = Service(EDGE_DRIVER_PATH)
    driver = webdriver.Edge(service=service, options=options)
    driver.set_page_load_timeout(60)
    return driver

# ========== 加载 / 保存 Cookie ==========
def save_cookies(driver, path=COOKIES_FILE):
    pickle.dump(driver.get_cookies(), open(path, "wb"))
    print(f"[+] Cookie 已保存到 {path}")

def load_cookies(driver, path=COOKIES_FILE):
    cookies = pickle.load(open(path, "rb"))
    for cookie in cookies:
        # Edge 对部分字段要求严格，移除不兼容字段
        cookie.pop('sameSite', None)
        try:
            driver.add_cookie(cookie)
        except Exception as e:
            print("添加 cookie 失败：", e)
    print("[+] Cookie 已加载")

# ========== 模拟用户行为 ==========
def human_scroll(driver):
    """随机滚动页面，模拟人工浏览，处理 scrollHeight 为 None 的情况"""
    height = driver.execute_script("return document.body.scrollHeight")
    if height is None or height < 400:
        # 如果获取不到高度或高度异常，设置一个默认高度
        height = 1000
    for i in range(3):
        pos = random.randint(200, max(height - 200, 200))
        driver.execute_script(f"window.scrollTo(0, {pos});")
        time.sleep(random.uniform(1.5, 3.0))
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    time.sleep(random.uniform(2, 3))

# ========== 保存当前页面 HTML ==========
def save_current_html(driver, filename=None):
    if not filename:
        filename = time.strftime("page_%Y%m%d_%H%M%S.html")
    filepath = os.path.join(SAVE_DIR, filename)
    html = driver.page_source
    with open(filepath, "w", encoding="utf-8") as f:
        f.write(html)
    print(f"[+] 已保存 HTML：{filepath}")

# ========== 切换到最新标签页 ==========
def auto_switch_to_newest_tab(driver):
    handles = driver.window_handles
    if len(handles) > 1:
        driver.switch_to.window(handles[-1])
        print("[*] 已切换到最新标签页。")

# ========== 自动翻页并保存 ==========
def auto_flip_and_save(driver, num_pages=5):
    """
    自动翻页并保存指定数量页面的HTML
    """
    auto_switch_to_newest_tab(driver)
    for i in range(1, num_pages + 1):
        print(f"正在处理第 {i} 页...")
        human_scroll(driver)
        save_current_html(driver, f"page_{i}.html")
        try:
            # 显式等待分页区加载
            wait = WebDriverWait(driver, 10)
            next_btn = wait.until(
                EC.presence_of_element_located(
                    (By.XPATH, "//*[contains(text(),'下一页')]")
                )
            )

            # 滚动到按钮位置
            driver.execute_script("arguments[0].scrollIntoView(true);", next_btn)
            time.sleep(random.uniform(1.5, 2.5))

            # 如果父元素是 a/button，就点击父元素
            parent = next_btn.find_element(By.XPATH, "./ancestor-or-self::*[self::a or self::button]")
            if parent:
                parent.click()
            else:
                # 否则用 JS 模拟点击
                driver.execute_script("arguments[0].click();", next_btn)

            wait_time = random.uniform(5, 8)
            print(f"[+] 已点击‘下一页’，等待 {wait_time:.1f} 秒加载...")
            time.sleep(wait_time)

        except Exception as e:
            print(f"[!] 翻页失败或已到最后一页：{e}")
            break

# ========== 主流程 ==========
def main():
    driver = init_driver()
    driver.get("https://www.taobao.com/")
    time.sleep(5)

    # 如果存在 cookie，尝试自动登录
    if os.path.exists(COOKIES_FILE):
        print("[*] 检测到已保存的 Cookie，尝试自动登录...")
        driver.delete_all_cookies()
        load_cookies(driver)
        driver.refresh()
        time.sleep(5)
        print("[*] 若登录状态正常，请继续。若未登录，请手动登录。")

    print("""
==============================
操作提示：
1. 若尚未登录，请在浏览器中手动登录淘宝。
2. 登录成功后在终端输入：login
   程序会自动保存 cookie。
3. 进入搜索结果页后在终端输入：save
   程序会保存当前渲染后的 HTML。
4. 输入 exit 退出。
==============================
""")

    while True:
        cmd = input(">>> 输入指令 (login/save/start/exit)：").strip().lower()
        if cmd == "login":
            save_cookies(driver)
        elif cmd == "save":
            try:
                human_scroll(driver)
            except Exception as e:
                print(f"滚动模拟出错，但将继续保存页面。错误：{e}")
            save_current_html(driver)
        elif cmd == "start":
            try:
                num_pages = int(input("请输入要保存的页数（默认5）：") or "5")
            except ValueError:
                print("输入无效，使用默认页数5。")
                num_pages = 5
            auto_flip_and_save(driver, num_pages)
        elif cmd == "exit":
            print("程序结束，关闭浏览器。")
            driver.quit()
            break
        else:
            print("无效指令，请输入 login/save/start/exit")

if __name__ == "__main__":
    main()
```

**save2csv.py**

```python
import os
import csv
from bs4 import BeautifulSoup
import re

# 文件夹路径
folder_path = './saved_pages'
output_csv = 'products.csv'

# CSV 表头
fields = [
    '商品标题', '封面链接', '价格', '付款人数', '省份', '城市',
    '支持的服务', '旗舰店昵称', '旗舰店tag'
]

all_products = []

def parse_sales(sales_text):
    """保留原始付款人数字符串"""
    if not sales_text:
        return ''
    return sales_text.strip()

# 遍历文件夹中的所有 HTML 文件
for filename in os.listdir(folder_path):
    if filename.endswith('.html'):
        with open(os.path.join(folder_path, filename), 'r', encoding='utf-8') as f:
            soup = BeautifulSoup(f, 'html.parser')
            # 找到所有商品的容器，这里改为通用 div 包含标题的结构
            items = soup.find_all('div', class_=re.compile(r'title--'))
            for item in items:
                # 商品标题
                title = item.get('title') or item.get_text(strip=True)

                # 封面图
                img_tag = item.find_previous('img')
                img_url = img_tag['src'] if img_tag else ''

                # 价格
                price_parent = item.find_next('div', class_=re.compile(r'priceWrapper--'))
                if price_parent:
                    price_int_tag = price_parent.find('div', class_=re.compile(r'priceInt--'))
                    price_float_tag = price_parent.find('div', class_=re.compile(r'priceFloat--'))
                    price = f"{price_int_tag.get_text(strip=True) if price_int_tag else ''}{price_float_tag.get_text(strip=True) if price_float_tag else ''}"

                    # 付款人数
                    sales_tag = price_parent.find('span', class_=re.compile(r'realSales--'))
                    sales = parse_sales(sales_tag.get_text(strip=True) if sales_tag else '')

                    # 省份和城市
                    area_tags = price_parent.find_all('div', class_=re.compile(r'procity--'))
                    province = area_tags[0].get_text(strip=True) if len(area_tags) > 0 else ''
                    city = area_tags[1].get_text(strip=True) if len(area_tags) > 1 else ''
                else:
                    price = sales = province = city = ''

                # 支持的服务
                services_parent = item.find_next('div', class_=re.compile(r'subIconWrapper--'))
                services_tags = services_parent.find_all('span') if services_parent else []
                services = ','.join([s.get_text(strip=True) for s in services_tags]) if services_tags else ''

                # 店铺信息
                shop_parent = item.find_next('a', class_=re.compile(r'shopName--'))
                shop_name_tag = shop_parent.find('span', class_=re.compile(r'shopNameText--')) if shop_parent else None
                shop_tag_tag = shop_parent.find('span', class_=re.compile(r'shopTagText--')) if shop_parent else None
                shop_name = shop_name_tag.get_text(strip=True) if shop_name_tag else ''
                shop_tag = shop_tag_tag.get_text(strip=True) if shop_tag_tag else ''

                all_products.append([
                    title, img_url, price, sales, province, city,
                    services, shop_name, shop_tag
                ])

# 保存到 CSV
with open(output_csv, 'w', newline='', encoding='utf-8-sig') as f:
    writer = csv.writer(f)
    writer.writerow(fields)
    writer.writerows(all_products)

print(f"解析完成，已保存 {len(all_products)} 条商品到 {output_csv}")
```

## 运行效果

![程序运行](../assets/taobaocrawler_2.jpg)

![效果展示](../assets/taobaocrawler_1.png)

