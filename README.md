信用卡欺诈检测项目步骤
1. 数据探索	Pandas（清洗、可视化）	查看类别分布、特征分布、相关性
2. 基线模型	sklearn 分类（逻辑回归）	训练、预测、用 F1/AUC 评估
3. 处理不平衡	无监督异常检测（高斯分布）或过采样	尝试 SMOTE 或调整 class_weight
4. 改进模型	随机森林 / XGBoost	对比不同模型性能

5. 


豆瓣爬取top250信息
1,爬虫四件套（requests, BeautifulSoup, pandas, time）以及正则
  import requests          # 发送 HTTP 请求
  from bs4 import BeautifulSoup  # 解析 HTML
  import pandas as pd      # 保存数据为 CSV
  import time              # 延时
  import re                # 正则表达式
2,定义请求头 headers（反爬必备） 
  headers = {
    'User-Agent': '...',   # 模拟浏览器
    'Accept': '...',
    'Accept-Language': '...',
    'Referer': '...',      # 告诉服务器从哪来
    'Cookie': '...'        # 身份凭证
    
     User-Agent 和 Referer 是基础，Cookie 可选但有时必要。
3,发送请求并获取网页（异常处理模板）
  def get_page(url):
    try:
        r = requests.get(url, headers=headers, timeout=15)
        r.raise_for_status()   # 状态码不是 200 则抛出异常
        r.encoding = 'utf-8'
        return r.text
    except Exception as e:
        print(f'请求失败: {e}')
        return None
  
  requests.get(url, headers=headers, timeout=...)
  r.raise_for_status()
  r.encoding = 'utf-8'
  try...except... 捕获异常，返回 None。
4,解析 HTML 的固定步骤
  soup = BeautifulSoup(html, 'html.parser')
  items = soup.find_all('div', class_='某个类名')
  
  BeautifulSoup(html, 'html.parser')
  find_all('标签', class_='类名') 查找所有符合条件的元素
  find('标签', class_='类名') 查找第一个
5,从元素中提取文本和属性
  tag = item.find('span', class_='title')
  if tag:
    text = tag.text          # 或 tag.get_text(strip=True)
    
  判断元素是否存在（if tag:）避免 NoneType 错误。
  用 .text 或 .get_text(strip=True) 获取文本内容。
6,正则表达式匹配数字或年份
  import re
  pattern = re.compile(r'\d+人评价')     # 编译正则
  match = re.search(r'(\d{4})', text)   # 搜索四位数字
  if match:
    year = match.group(1)             # 提取括号内捕获的内容
    
  re.compile(r'正则表达式') 提前编译。
  re.search(pattern, string) 搜索第一次匹配。
  group(1) 获取第一个括号内的内容。
  常用正则：\d+ 数字，\d{4} 四位数字。
7,提取嵌套元素（链式查找）
  info_ps = item.find('div', class_='bd').find_all('p')
  if info_ps:
    first_p = info_ps[0].text.strip()
  链式调用：先找父容器 .find(...)，再在结果上继续 .find_all(...)。
  取列表第一个元素用 [0]。
8,数据存储到列表并转 DataFrame
  movie_list = []
  movie_list.append({
    '列名1': 值1,
    '列名2': 值2
  })
df = pd.DataFrame(movie_list)
df.to_csv('文件名.csv', index=False, encoding='utf-8-sig')
  每条记录用字典表示，多个字典放入列表。
  pd.DataFrame(list_of_dicts) 直接转为表格。
  to_csv 中 index=False 去掉行号，encoding='utf-8-sig' 让 Excel 正常打开。
9,循环与延时控制
  for start in range(0, 250, 25):    # 0, 25, 50, ..., 225
    url = base_url.format(start)
    html = get_page(url)
    if html:
        # 解析
        time.sleep(2)   # 暂停 2 秒
    else:
        break
  range(start, stop, step) 生成等差数列。
  字符串格式化 url.format(变量)。
  每次请求后 time.sleep(秒) 防止反爬。
10,主程序入口标准写法
  if __name__ == '__main__':
    main()
  使得模块既可以作为脚本运行，也可以被导入而不自动执行。

