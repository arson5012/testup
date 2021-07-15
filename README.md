# :tiger: B 리그_범 올라온다
KIEE 2021 Mini Drone Flight Competition B_League Bumup  
2021 미니드론 B리그 범올라온다 입니다

---
<!-------------------------------------------------------------Part 1------------------------------------------------------------------------------------------>

## 대회 진행 전략

 * **파란색 링 검출 및 중점 찾기**
 1. 파란색 HSV 설정
 2. 컬러 BGR 영상을 HSV로 변환후 설정된 HSV값에 의해 이진화된 이미지 출력

 3. 인식된 파란색 링의 중점 찾기   
     > cv2.connectedComponentsWithStats 함수를 사용하여 중점을 찾음\
     >![HSV](https://1.bp.blogspot.com/-kiX-x_qotU0/YOuRpGKUUaI/AAAAAAAAACc/E5F_1lZNU0MX8pnXHPeVUl4f99RfzbKJwCLcBGAsYHQ/w394-h308/2%25EB%258B%25A8%25EA%25B3%2584%2B%25281%2529.PNG)

 * **드론 이동**
     > 드론 카메라의 화면 중점과 화면 안에서 검출된 파란색 링의 픽셀들 중점을 비교
 1. 상승 & 하강 제어 \
    ![상하](https://1.bp.blogspot.com/-1Eo_hVjjndA/YOfqAbu7W6I/AAAAAAAAACI/40jUd1XetfQeUTMNQ8u-SoAodLHzSutkACLcBGAsYHQ/w732-h298/%25EC%2583%2581%25ED%2595%2598.jpg)
 1. 좌 & 우 이동 제어\
    ![좌우](https://1.bp.blogspot.com/-uxoMlI81qmE/YOfqASY_C6I/AAAAAAAAACE/xNqdFpkVSuIdSW_1BumhjbCJf4s--4DwACLcBGAsYHQ/w737-h300/%25EC%25A2%258C%25EC%259A%25B0.jpg)
 * **빨간색 / 보라색 색상 검출**
    > 인식된 빨간색 표식의 픽셀 수에 따라 링의 거리 추정  
      ![링거리](https://lh3.googleusercontent.com/-NxNQWg7fGx4/YOfEdIFLnDI/AAAAAAAAAB0/YXxvg2rDA0II3rLnqSUkZmq9gybzF6l3ACLcBGAsYHQ/w456-h234/rr.jpg)
 1. 빨간색 HSV 설정
    > 빨간색 표식을 인식하면 회전  
      ![적색](https://1.bp.blogspot.com/-ba4HRclZpec/YOuQCt66HoI/AAAAAAAAACU/NXbd0MnnYIQa-wPthcl6M4vk0pqbiaWHwCLcBGAsYHQ/w722-h268/KakaoTalk_20210709_142837811_02.png)
 1. 보라색 HSV 설정
    > 보라색 표식을 인식하면 착지  
      ![보라색](https://1.bp.blogspot.com/-VqRvy_duZqM/YOwWi2mvi6I/AAAAAAAAACs/WqJbGqco1zgAITkajT70kvVwxIJwMWdhACLcBGAsYHQ/w722-h271/pur.png)
 ---

 <!-------------------------------------------------------------Part 2------------------------------------------------------------------------------------------>
 ## 알고리즘 설명
 <center>
     <img src="https://1.bp.blogspot.com/-B_Op5WMY530/YO492SI13jI/AAAAAAAAADU/5n2Yke5EYTEMR-z4itDKXyFYuEDTmLljwCLcBGAsYHQ/w885-h498/KakaoTalk_20210714_102416927.png">
 </center>

 1. 드론 연결 및 이륙
     > sendTakeOff() 함수를 사용한 드론을 이륙한다.
 2. 영상수신
     > for frame in camera.capture_continuous(rawCapture, format="bgr", use_video_port=True):  
       실시간 영상을 수신한다.
 3. 전처리  
     > 입력영상에서 원하는 색상만을 사용하기 위한 과정.    
       파란색의 링, 1, 2차의 빨간색 표식 그리고 3차의 보라색 표식의 HSV 값을 미리 저장한 후,  
       원본영상을 HSV 색영상으로 변환, 이진화하여 필요한 색상만을 찾는다.  

 4. 링 중점 좌표 계산  
      > 드론이 링을 통과할 수 있도록 입력영상에서 링의 중점 좌표를 찾는다. 

 5. 드론 위치 제어  
      > 앞서 찾은 링의 중점좌표로 드론의 현재 위치를 판단하고     
        링의 중앙에 드론이 위치하도록 좌우상하를 제어한다.  
        
 6. 직진 및 표식 확인  
       > 드론의 위치가 링의 중앙에 위치한다고 판단되면 표식을 인식할 수 있도록 직진한다.  
 
 7. 표식과의 거리 판단  
      > 입력영상에서 표식색상의 픽셀개수를 통해 표식과 드론간의 거리를 판단한다.  
        링 통과 전 표식의 중점을 재탐색하고 그 중점을 기준으로 드론의 좌우상하를 제어한 후, 직진하여 링을 통과한다.      
        
   
 8. 회전 및 착륙  
      > 링 통과 후, 빨간 표식이 확인되면 드론을 회전시키고 보라색 표식이 확인되면 드론을 착륙시킨다.  

 ---
 <!-------------------------------------------------------------Part 3------------------------------------------------------------------------------------------>
 ## 소스코드 설명

 **표식 픽셀개수 계산**
 ```python
         R = np.sum(sum_red == 255, axis=None)
         P = np.sum(bin_p == 255, axis=None)
         B = np.sum(bin_b == 255, axis=None)
 ```
 **링 중점 계산**
 ```python
         # 파란색 중심 찾기
         nlabels, labels, stats, centroids = cv2.connectedComponentsWithStats(bin_b)
         for ida, centroids in enumerate(centroids):
             if stats[ida][0] == 0 and stats[ida][1] == 0:
                 continue
             if np.any(np.isnan(centroids)):
                 continue
             x, y, width, height, area = stats[ida]
             if (area > 300):
                 centerX, centerY = int(centroids[0]), int(centroids[1])
 ```
 **링 검출 후 위치제어**
 ```python
        elif flag == 0: 
        # 파란색 링 검출
        
             if B < 100 : 
             # 검출 되지 않으면 왼쪽으로 이동
                 drone.sendControlPosition16(0, 10, 0, 7, 0, 0)
                 sleep(5)
                 round(real_height, 2)
                 drone.sendControlPosition(0, 0, round(x_height - real_height, 2), 8, 0, 0) 
                 # 드론 현재 높이에 따라 고도 맞추기
                 sleep(5)

             if B > 100:
             # 파란색 링 검출 시 중점 좌표 계산 / 비교
             # 상하좌우 일치하면 직진
                 if y_flag == False:
                     if centerY > 250 and centerY < 280:
                         y_flag = True
                         print('y_flag = TRUE')
                     # 상하 조정
                     elif centerY > 280:
                         print('down')
                         drone.sendControlPosition(0, 0, -0.07, 0.5, 0, 0)
                         sleep(0.5)
                     elif centerY < 250:
                         print('up')
                         drone.sendControlPosition(0, 0, 0.07, 0.5, 0, 0)
                         sleep(0.5)
                 if y_flag == True:
                     if centerX > 305 and centerX < 335:
                         #y_flag = False
                         print("Forward")
                         y_flag = False
                         
                         if purple < 2:
                             flag = 1
                         elif purple == 2:
                             flag = 2
                             
                     # 좌우 조정
                     elif centerX > 335:
                         drone.sendControlPosition(0, -0.07, 0, 0.5, 0, 0)
                         print('right')
                         sleep(0.5)
                     elif centerX < 305:
                         drone.sendControlPosition(0, 0.07, 0, 0.5, 0, 0)
                         print('left')
                         sleep(0.5)

 ```

 **표식 검출 후, 드론 이동 제어**
 ```python
         if flag == 2:
         # 보라색 표식 검출
             if P < 200:
             # 검출 시 이동
                 print("purple_move")
                 drone.sendControlPosition(0.3, 0, 0, 0.2, 0, 0)
                 sleep(1)

             elif P > 200 and P < 2000:
                 nlabels3, labels3, stats3, centroids3 = cv2.connectedComponentsWithStats(bin_p)
                 for ida, centroids3 in enumerate(centroids3):
                     if stats3[ida][0] == 0 and stats3[ida][1] == 0:
                         continue
                     if np.any(np.isnan(centroids3)):
                         continue
                     x3, y3, width3, height3, area3 = stats3[ida]
                     if (area3 > 10):
                         center3X, center3Y = int(centroids3[0]), int(centroids3[1])

                 if purple_flag == False:
                     if center3Y > 245 and center3Y < 285:
                         purple_flag = True
                         print('purple = TRUE')
                     # 상하 조정
                     elif center3Y > 285:
                         print('purple down')
                         drone.sendControlPosition(0, 0, -0.05, 0.1, 0, 0)
                         sleep(0.5)
                     elif center3Y < 245:
                         print('purple up')
                         drone.sendControlPosition(0, 0, 0.05, 0.1, 0, 0)
                         sleep(0.5)
                 elif purple_flag == True:
                     if center3X > 305 and center3X < 335:
                         # y_flag = False
                         print("Purple Forward")
                         drone.sendControlPosition(0.3, 0, 0, 0.2, 0, 0)
                         sleep(1)
                     # 좌우 조정
                     elif center3X > 335:
                         drone.sendControlPosition(0, -0.05, 0, 0.1, 0, 0)
                         print('purple right')
                         sleep(0.5)
                     elif center3X < 305:
                         drone.sendControlPosition(0, 0.05, 0, 0.1, 0, 0)
                         print('purple left')
                         sleep(0.5)
             if P > 2000:
             # 보라색 표식 확인 후 착륙
                 print("Landing")
                 drone.sendLanding()
                 drone.close()
                 break

         elif flag == 1 :
         # 빨간색 표식 검출
             if R < 200:
                 print("red_move")
                 drone.sendControlPosition(0.3, 0, 0, 0.2, 0, 0)
                 sleep(1)

             elif R > 200 and R < 2000:
                 nlabels2, labels2, stats2, centroids2 = cv2.connectedComponentsWithStats(sum_red)
                 for ida, centroids2 in enumerate(centroids2):
                     if stats2[ida][0] == 0 and stats2[ida][1] == 0:
                         continue
                     if np.any(np.isnan(centroids2)):
                         continue
                     x2, y2, width2, height2, area2 = stats2[ida]
                     if (area2 > 10):
                         center2X, center2Y = int(centroids2[0]), int(centroids2[1])

                 if red_flag == False:
                     if center2Y > 245 and center2Y < 285:
                         red_flag = True
                         print('red_flag = TRUE')
                     # 상하 조정
                     elif center2Y > 285:
                         print('red down')
                         drone.sendControlPosition(0, 0, -0.05, 0.1, 0, 0)
                         sleep(0.5)
                     elif center2Y < 245:
                         print('red up')
                         drone.sendControlPosition(0, 0, 0.05, 0.1, 0, 0)
                         sleep(0.5)
                 elif red_flag == True:
                     if center2X > 280 and center2X < 360:
                         #y_flag = False
                         print("Red Forward")
                         drone.sendControlPosition(0.3, 0, 0, 0.4, 0, 0)
                         sleep(1)
                     # 좌우 조정
                     elif center2X > 360:
                         drone.sendControlPosition(0, -0.05 , 0, 0.1, 0, 0)
                         print('red right')
                         sleep(0.5)
                     elif center2X < 280:
                         drone.sendControlPosition(0, 0.05, 0, 0.1, 0, 0)
                         print('red left')
                         sleep(0.5)


             if R > 2000:
             # 빨간색 표식 확인 후 90도 좌회전
                 print("curve")
                 drone.sendControlPosition16(0, 0, 0, 0, 90, 17)  
                 sleep(7) 
   
                 drone.sendControlPosition16(10, 0, 0, 4, 0, 0) 
                 sleep(7) 
                 
                 round(real_height, 2)
                 drone.sendControlPosition(0, 0, round(x_height-real_height, 2), 8, 0, 0)
                 # 드론 현재 높이에 따라 고도 맞추기
                 
                 sleep(5)
                 drone.sendControlPosition16(0, -15, 0, 4, 0, 0)
                 # 오른쪽으로 이동

                 sleep(8)
                 flag = 0
                 
                 red_flag = False
                 purple += 1
                 # 보라색 표식 검출 카운트 추가
             


 ```

