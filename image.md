import cv2
import numpy as np

cap = cv2.VideoCapture('E:\\4_1.mp4')

if not cap.isOpened():
    print("Cannot open video file")
    exit()
while True:
    # 讀取一帧
    ret, frame = cap.read()

    if not ret:
        print("Cannot receive frame")
        break
    # 將每一帧转换为灰度图像
    gray_frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    # 二值化处理
    _, binary_frame = cv2.threshold(gray_frame, 121, 255, cv2.THRESH_BINARY)

    # 检测轮廓
    contours, _ = cv2.findContours(binary_frame, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
    sorted_contours = sorted(contours, key=cv2.contourArea, reverse=True)
    # 找到最大的轮廓
    max_contour = sorted_contours[1]

    # 選擇最大輪廓中的每一點
    all_points = np.concatenate(max_contour)
    # 根據 XY 的相加找到左上角
    left_top_index = np.argmin(np.sum(all_points, axis=1))
    left_top_point = tuple(all_points[left_top_index])

    # 根據 XY 的相加找到右下角
    right_bottom_index = np.argmax(np.sum(all_points, axis=1))
    right_bottom_point = tuple(all_points[right_bottom_index])

    # 根據 X-Y 的相加找到左下角
    left_bottom_index = np.argmax(np.diff(all_points, axis=1))
    left_bottom_point = tuple(all_points[left_bottom_index])

    # 根據 X-Y 的相加找到右上角
    right_top_index = np.argmin(np.diff(all_points, axis=1))
    right_top_point = tuple(all_points[right_top_index])
    if not abs(left_top_point[0] - left_bottom_point[0]) < 10:
        continue
    if not abs(right_top_point[0] - right_bottom_point[0]) < 10:
        continue
    # 透視變換矩陣
    paper_corners = np.float32([left_top_point, right_top_point, right_bottom_point, left_bottom_point])
    output_size = (226, 372)  # 新的目標尺寸
    output_corners = np.array(
        [[0, 0], [output_size[0] - 1, 0], [output_size[0] - 1, output_size[1] - 1], [0, output_size[1] - 1]],
        dtype=np.float32)
    perspective_matrix = cv2.getPerspectiveTransform(paper_corners, output_corners)

    # 進行透視變換
    warped_frame = cv2.warpPerspective(frame, perspective_matrix, output_size)

    # 使用霍夫變換檢測直線
    edges = cv2.Canny(warped_frame, 50, 150, apertureSize=3)
    lines = cv2.HoughLines(edges, 1, np.pi / 180, threshold=100)

    if lines is not None:
        rho, theta = lines[0][0]
        angle = theta * (180 / np.pi)
        # 判斷是否是垂直線，且旋轉矩形角度在合理範圍內

        if not (0 <= angle <= 2):
            continue

        # 計算旋轉矩形的角度
        if 0 <= angle <= 2:
            # 計算旋轉矩形的角度
            rotated_rect = cv2.minAreaRect(max_contour)
            rotated_rect_angle = rotated_rect[2]
            if not (0 <= rotated_rect_angle <= 3):
                continue  # 如果不是垂直線，跳過這一幀

    top_left = (54, 156)
    bottom_right = (226, 372)
    original_image = warped_frame[top_left[1]:bottom_right[1], top_left[0]:bottom_right[0]]
    original_image = cv2.cvtColor(original_image, cv2.COLOR_BGR2GRAY)
    resized_image = cv2.resize(original_image, (688, 864))
    cv2.imshow('oxxostudio', resized_image)
    if cv2.waitKey(0) == ord('Q'):
        break



cap.release()
cv2.destroyAllWindows()
