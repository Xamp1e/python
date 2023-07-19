# python
python_scripts_for_bioinfomatics
import csv
import requests
from bs4 import BeautifulSoup

def search_google_scholar(species):
    query = species + ' recombination rates female male cm'
    url = 'https://scholar.google.com/scholar'
    params = {
        'q': query,
        'hl': 'en',
        'as_sdt': '0,5'
    }

    response = requests.get(url, params=params)
    response.raise_for_status()

    soup = BeautifulSoup(response.text, 'html.parser')
    results = soup.find_all('h3', class_='gs_rt')

    articles = []
    for result in results:
        article_title = result.get_text().strip()
        articles.append(article_title)

    return articles

def save_results_to_csv(file_name, results):
    with open(file_name, 'w', newline='', encoding='utf-8') as file:
        writer = csv.writer(file)
        writer.writerow(['Species', 'Article Title'])
        writer.writerows(results)

# 读取物种列表文件
species_list_file = 'species_list.txt'
with open(species_list_file, 'r', encoding='utf-8') as file:
    species_list = [line.strip() for line in file]

# 搜索并保存结果
output_file = 'result.csv'
all_results = []
species_set = set()  # 记录已处理的物种名
for species in species_list:
    species_lower = species.lower()
    if species_lower not in species_set:
        results = search_google_scholar(species)
        if results:
            article_title = results[0]  # 只获取第一个文章
            all_results.append([species, article_title])
        species_set.add(species_lower)

save_results_to_csv(output_file, all_results)
