# B 리그_범 올라온다
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
    >![HSV](https://1.bp.blogspot.com/-wScFOulU8-c/YOfC9pS9tDI/AAAAAAAAABU/DA0k8E1LH1AIUrZydIanKJyCfPLj5xmNQCLcBGAsYHQ/w415-h332/2%25EB%258B%25A8%25EA%25B3%2584.PNG)
  
* **드론 이동**
    > 드론 카메라의 화면 중점과 화면 안에서 검출된 파란색 링의 픽셀들 중점을 비교
1. 상승 & 하강 제어 \
   ![상하](https://1.bp.blogspot.com/-1Eo_hVjjndA/YOfqAbu7W6I/AAAAAAAAACI/40jUd1XetfQeUTMNQ8u-SoAodLHzSutkACLcBGAsYHQ/w732-h298/%25EC%2583%2581%25ED%2595%2598.jpg)
1. 좌 이동 & 우 이동 제어\
   ![좌우](https://1.bp.blogspot.com/-uxoMlI81qmE/YOfqASY_C6I/AAAAAAAAACE/xNqdFpkVSuIdSW_1BumhjbCJf4s--4DwACLcBGAsYHQ/w737-h300/%25EC%25A2%258C%25EC%259A%25B0.jpg)
* **빨간색 / 보라색 색상 검출**
   > 인식된 빨간색 표식의 픽셀에 따라 링의 거리 추정  
     ![링거리](https://lh3.googleusercontent.com/-NxNQWg7fGx4/YOfEdIFLnDI/AAAAAAAAAB0/YXxvg2rDA0II3rLnqSUkZmq9gybzF6l3ACLcBGAsYHQ/w456-h234/rr.jpg)
1. 빨간색 HSV 설정
   > ![적색](https://1.bp.blogspot.com/-ba4HRclZpec/YOuQCt66HoI/AAAAAAAAACU/NXbd0MnnYIQa-wPthcl6M4vk0pqbiaWHwCLcBGAsYHQ/w722-h268/KakaoTalk_20210709_142837811_02.png)
1. 보라색 HSV 설정
   > 보라색 표식을 인식하면 착지  
---

<!-------------------------------------------------------------Part 2------------------------------------------------------------------------------------------>
## 알고리즘 설명
![블록도](https://lh3.googleusercontent.com/proxy/JcYnsKKvxbL00xLeGRfyjEL7p4-rPjAJVZdgNq2aOgFpIipVjDtU4p6salj1-5Ak4U6Nxdq_jgMsX85jy2_-TeR1NPcpQ070_3ZMWdtP5wQcPnk89JIAZQXM6ADR76nLALExjyQEGgi7sQF3FyhK9o3UaoLoDdd1)

1. 드론 연결 및 이륙
    > sendTakeOff() 함수를 사용한 드론 이륙
2. 영상수신
    > for frame in camera.capture_continuous(rawCapture, format="bgr", use_video_port=True):  
      실시간 영상 수신
3. 전처리
    > 입력영상에서 원하는 색상만을 사용하기 위한 과정
      파란색의 통과할 링, 1,2차의 빨간 표식 그리고 3차의 보라 표식의 HSV값을 미리 저장한 후, 
      원본영상을 HSV 색영상으로 변환, 이진화하여 필요한 색상만을 찾는다.
      
4. 링 위치 파악
     > 빨간 표식 픽셀에 따라 링의 거리를 판단하고 1미터 보다 가깝다면 빨간 표식을 중점으로 계산하고,  
       1미터 보다 멀다면 링의 중점을 찾도록 한다
   
5. 링 중점 좌표 계산  
     > 링이 1미터 보다 멀다면 드론이 링을 통과할 수 있도록 입력영상에서 링의 중점 좌표를 찾는다. 

<!---6. 링 통과 후 픽셀개수 계산
    > 드론과 표식과의 거리를 판단하기 위한 방법으로  
      드론이 표식과 가까워지면 이진화된 입력영상에서 들어오는 픽셀의 개수가 커진다.
      이를 이용하여 드론이 링을 통과한 후 표식과 가까워졌는지를 판단한다.      -->
      
6. 드론 위치 제어  
     > 앞서 찾은 링의 중점좌표로 드론의 현재 위치를 판단하고 링의 중앙에 드론이 위치하고록 제어한다.  
       중앙에 위치되면 직진한다.
  
7. 직진 후, 표식 확인  
     > 드론이 직진한 후에 입력영상에서 표식색상의 픽셀개수를 통해 표식을 확인한다.\
       찾지 못하면 상하,전후진 이동을 통해 표식을 탐색한다. 
  
8. 회전 및 착륙  
     > 빨간 표식이 확인되면 드론을 회전시키고 보라색 표식이 확인되면 드론을 착륙시킨다.  

---

<!-------------------------------------------------------------Part 3------------------------------------------------------------------------------------------>
## 소스코드 설명

**표식 픽셀개수 계산**
```python
        R = np.sum(sum_red == 255, axis=None)
        P= np.sum(bin_p == 255, axis=None)
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
**드론 위치제어**
```python
        # 코드 넣기
        
```

**표식 검출 후, 드론 이동 제어**
```python
        # 코드 넣기
        
```
