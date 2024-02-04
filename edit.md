import csv
import tkinter as tk
from tkinter import messagebox

def calculate_total_votes(csv_file_path):
    # 讀取CSV檔案
    with open(csv_file_path, 'r', encoding='utf-8') as file:
        # 創建CSV讀取器
        reader = csv.reader(file)
        
        # 跳過標題行
        next(reader, None)

        # 初始化總票數
        total_votes = 0

        # 遍歷每一行，提取領票數並加總
        for row in reader:
            # 假設領票數在第一列
            ticket_count = int(row[1])
            total_votes += ticket_count
    return total_votes
    
def read_csv_file(csv_file_path):
    rows = []  # 用來存儲每一行資訊的列表

    with open(csv_file_path, 'r', encoding='utf-8') as file:
        reader = csv.reader(file)

        # 跳過標題行
        header = next(reader, None)

        if header:
            # 遍歷每一行，將每一行的資訊存儲在一個字典中
            for line_number, row in enumerate(reader, start=1):
                row_data = {'行號': line_number}
                for i in range(len(header)):
                    row_data[header[i]] = row[i]
                rows.append(row_data)
    return rows
    
csv_file_path = "E:\Python\output.csv"
csv_rows = read_csv_file(csv_file_path)
calculate_count=calculate_total_votes(csv_file_path)
root = tk.Tk()
root.withdraw()
count = int(input("請輸入領票數: "))
art_count=[]
x=1
for row in csv_rows:
    c=int(input(f"輸入{x}號候選人的票數: "))
    art_count.append(c)
    x+=1

if count == calculate_count:
    print("輸入的領票數和計算的總票數相等")
    print(f"輸入的領票數:{count} 計算的總票數: {calculate_count}")
else:
    print("輸入的領票數和計算的總票數不相等")
    print(f"輸入的領票數:{count} 計算的總票數: {calculate_count}")

for i, row in enumerate(csv_rows):
    candidate_name = row["Text"]  # 假設候選人名稱在 CSV 的 "候選人" 欄位中
    csv_vote_count = int(row["Count"])
    user_input_vote_count = art_count[i]
    print(f"{candidate_name}：語音偵測的得票數 {csv_vote_count}，使用者輸入得票數 {user_input_vote_count}")
    if csv_vote_count!=user_input_vote_count:
        messagebox.showwarning("錯誤錯誤!!", f"{candidate_name} 有錯誤，請人工盤查")
        print(f"{candidate_name} 有錯誤，請人工盤查")
