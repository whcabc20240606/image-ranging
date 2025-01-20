# image-ranging
# -*- coding: cp936 -*-
import cv2
import math
import random
import numpy as np
from PIL import Image
import matplotlib.pyplot as plt


def pt_distance(pt1, pt2):
    dist = math.sqrt(pow(pt1[0]-pt2[0], 2) + pow(pt1[1]-pt2[1], 2))
    return dist


path = '../Date/splattering/super_extracted/extr 3.0x-b_frame225.png'
img = cv2.imread(path)
gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
ret, thresh = cv2.threshold(gray, 220, 255, cv2.THRESH_BINARY)
contours, hierarchy = cv2.findContours(thresh, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)
cv2.imshow('thresh', thresh)
print(contours)
print("Number of contours detected：", len(contours))
temp1 = img.copy()
# cv2.drawContours(temp1, contours, -1, (10, 10, 166), 1)
X = 9
Y = 12

areas = [cv2.contourArea(contour) for contour in contours]
sorted_indices = sorted(range(len(areas)), key=lambda k: areas[k], reverse=True)
top_three_indices = sorted_indices[:7]
if sorted_indices[0] in top_three_indices:
    del top_three_indices[0]
for i, contour in enumerate(contours):
    area = cv2.contourArea(contour)
    # print("Contour", i, "Area:", area)

    if i in top_three_indices:
        M = cv2.moments(contour)
        if M["m00"] == 0:
            continue
        cX = int(M["m10"] / M["m00"])
        cY = int(M["m01"] / M["m00"])
        cv2.putText(temp1, str(i), (cX, cY), cv2.FONT_HERSHEY_DUPLEX, 0.8, (35, 55, 255), 1)
        cv2.imshow('Numbers', temp1)

flag = False
minDist = 1000
minPt0 = (0, 0)
minPt1 = (0, 0)

for i in range(0, len(contours[X])):
  # print(len(contours[1]))
  pt1 = tuple(contours[X][i][0])
  min = 1000
  min_pt = (0, 0)
  min_point = (0, 0)

  for j in range(0, len(contours[Y])):
    pt2 = tuple(contours[Y][j][0])
    distance = pt_distance(pt1, pt2)
    if distance < min:
      min = distance
      min_pt = pt2
      min_point = pt1
  if min < minDist:
    minDist = min
    minPt0 = min_point
    minPt1 = min_pt

    contour_index1 = X
    target_contour1 = contours[contour_index1]
    contour_index2 = Y
    target_contour2 = contours[contour_index2]
    cv2.drawContours(img, [target_contour1], -1, (255, 0, 255), 1)
    cv2.drawContours(img, [target_contour2], -1, (255, 255, 0), 1)
  temp2 = img.copy()
  cv2.line(temp2, min_point, min_pt, (0, 255, 0), 1, cv2.LINE_AA)
  cv2.circle(temp2, min_point, 2, (255, 0, 0), -1, cv2.LINE_AA)
  cv2.circle(temp2, min_pt, 2, (0, 0, 255), -1, cv2.LINE_AA)
  cv2.imshow("Running", temp2)
  if cv2.waitKey(1) & 0xFF == 27:
    flag = True
    break
  if flag:
    break

font = cv2.FONT_HERSHEY_SIMPLEX
cv2.line(img, minPt0, minPt1, (0, 255, 0), 1, cv2.LINE_AA)
cv2.circle(img, minPt0, 2, (255, 0, 0), -1, cv2.LINE_AA)
cv2.circle(img, minPt1, 2, (0, 0, 255), -1, cv2.LINE_AA)
# cv2.putText(img, ("min=%0.3f" % minDist), (minPt1[0], minPt1[1]+15), font, 0.6, (0, 255, 0), 2)

image = Image.open(path)
dpi = image.info.get('dpi')
numbers = dpi
dpi_value = random.choice(numbers)
Distence = (2*minDist*25.4)/dpi_value
rounded = round(minDist, 3)
cv2.putText(img, ("min=%0.3fcm" % Distence), (minPt1[0], minPt1[1]+15), font, 0.6, (0, 255, 0), 2)

print('Shortest distance point coordinates：', [minPt0, minPt1])
print('Minimum pixel distance between target contours:', rounded)
print('Minimum graphed distance between target contours:{}mm'.format(round(Distence, 3)))
cv2.imshow('result', img)  # cv2.imshow()
cv2.imwrite('result.png', img)

cv2.waitKey(0)
cv2.destroyAllWindows()
