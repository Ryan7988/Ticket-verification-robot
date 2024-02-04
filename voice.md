import speech_recognition as sr
import csv

def Voice_To_Text(duration=20): 
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("請開始說話:")
        r.adjust_for_ambient_noise(source)
        audio = r.listen(source, phrase_time_limit=duration)
    try:
        Text = r.recognize_google(audio, language="zh-TW")
    except sr.UnknownValueError:
        Text = "無法翻譯"
    except sr.RequestError as e:
        Text = "無法翻譯{0}".format(e)
    return Text

csv_path = "E:\Python\output.csv"
header = ["Text", "Count"]
text_counts = {}

def compare_first_two_chars(a, b):
    return a[0] == b[0] and a[1] == b[1]
    
def compare_strings(str1, str2):
    common_count = sum(1 for char in set(str1) if char in str2)
    return common_count >= 2

def sort_text_counts(text_counts):
    sorted_counts = sorted(text_counts.items(), key=lambda x: x[0])
    return sorted_counts
    
with open(csv_path, "w", newline="", encoding="utf-8") as csv_file:
    csv_writer = csv.writer(csv_file)
    csv_writer.writerow(header)
    
    for i in range(5):
        Text = Voice_To_Text(10)
        match_found = False
        for existing_text, count in text_counts.items():
            if compare_first_two_chars(existing_text, Text) or compare_strings(existing_text, Text):
                text_counts[existing_text] += 1
                match_found = True
                break

        if not match_found:
            text_counts[Text] = 1
            
        # 將文件指針移動到文件的開頭，然後清空文件內容
        csv_file.seek(0)
        csv_file.truncate()

        # 重新寫入表頭
        csv_writer.writerow(header)

        sorted_counts = sort_text_counts(text_counts)

        # 寫入 text_counts 的內容
        for idx, (text, count) in enumerate(sorted_counts, start=1):
            csv_writer.writerow([text, count])
            print(f"{idx}. {text} - {count}")
   

print("已將語音轉換結果寫入 output.csv 文件")
