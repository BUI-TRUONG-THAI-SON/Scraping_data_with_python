import json
import requests
from bs4 import BeautifulSoup
base_url = 'https://vnexpress.net/phap-luat/ho-so-pha-an-p'
baibao_list = []
def extract_text(soup, selector):
    # ham nay dung de kiem tra xem có attribute như yêu cầu trích xuất không
    try: 
        return soup.select_one(selector).get_text(strip = True, separator = '')
    except AttributeError as err:
        # print("Bài báo không có dạng chữ")
        return None
def an_article(url, order, page_number):
    # lay thong tin cua bai bao cua trang thu page_number
    website = requests.get(f'{url}').text
    soup = BeautifulSoup(website, 'lxml')
    tieu_de = extract_text(soup, 'h1.title-detail')
    # Kiểm tra xem bài báo có dạng văn bản hay dạng podcast
    if tieu_de == None: # Nếu bài báo có dạng podcast
        tieu_de = extract_text(soup, 'h1.title-news')
        noi_dung = 'Bài báo có dạng podcast'
    else: # Nếu bài báo có có dạng văn bản 
        noi_dung = extract_text(soup, 'article.fck_detail')
    # Thu thap du lieu nhu yeu cau
    baibao_data = {
        'Bài báo: ': f"Trang {page_number}, số {order}",
        'title': tieu_de,
        'content': noi_dung,
        'url': url
    }
    baibao_list.append(baibao_data)
def scraping_page(duong_dan, order, page_number):
    # lay url cua trang thu page_number
    website = requests.get(f'{duong_dan}').text
    soup = BeautifulSoup(website, 'lxml')
    # xử lý bài báo hot đầu tiên
    title_hot = soup.find('article', class_ = 'item-news full-thumb article-topstory')
    # kiểm tra xem hết bài báo hot chưa, đồng nghĩa với việc nếu hết thì trang đó không còn bài báo để thu thập data
    if title_hot == None:
        return False
    url = title_hot.a['href']
    print(url)
    an_article(url, order, page_number)
    order+=1
    # xử lý bài báo phổ thông phía dưới bài báo hot
    title_thuong = soup.find_all('article', class_ = 'item-news thumb-left item-news-common')
    for title in title_thuong:
        url = title.a['href']
        print(url)
        an_article(url, order, page_number)
        order+=1
    return True
page_number = 1
order = 1
while True:
    duong_dan = base_url + str(page_number)
    print("Đang lấy dữ liệu " + duong_dan)
    if scraping_page(duong_dan, order, page_number):
       page_number += 1
    else:
        print(f'Trang {page_number} không có dữ liệu, đã hoàn thành việc lấy dữ liệu ở trang {page_number-1}')
        break
file_json = "vnexpress_phapluat_ho_so_pha_an.json"
with open(file_json, 'w', encoding='utf-8') as file:
    json.dump(baibao_list, file, ensure_ascii=False, indent=4)


