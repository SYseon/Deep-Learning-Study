# YOLO v1 (You Only Look Once,2016)

**Object detection**을 위해 설계된 **CNN** 구조의 알고리즘 

</br>

## 이전의 Object detection 방법

- DPM (Deformable Parts Model) : Sliding Window 방법 이용
 
- R-CNN : Region Proposal 방법 이용

  공통점: Bounding Box 찾고 Classification

  => YOLO는 두 과정을 single convolution network 하나로 합침
  
  </br>
  
  
## YOLO의 장점
  
**1 . Speed** 

  : **single convolution network**로 실행하므로 이전 모델보다 **빠르다**
  
  </br>
   
**2 . Less background mistakes**
  
  : 부분이 아닌 **전체 이미지**를 입력받으므로 background mistake가 적다
  
  </br>

**3 . Highly generalizable**

  : natural 이미지로 학습하고 artwork로 테스트해도 성능이 좋다.

  => **새로운 도메인**이나 예상치 못한 입력값 들어와도 **break down 확률 적다**
  
  </br>
  

단, 작은 물체는 잘 detect하지 못한다.

</br>


## Unified Detection

![dog](https://user-images.githubusercontent.com/33515697/45637624-ef4edc00-bae5-11e8-8d99-050315645272.png)

- 이미지를 S X S grid로 나눈다.

- object의 중심은 어떤 하나의 grid cell 안에 포함되고, 그 grid cell이 object 존재를 책임진다.

- 각 grid cell은 하나의 class를 예측한다.

- 각 grid cell은 B개의 Bounding box를 예측한다. Bounding box는 x,y,w,h,confidence로 구성된다.
  - (x,y) : Bounding box의 중심 좌표 (grid cell 기준 , 0과 1 사이값)
  - w,h : Bounding box의 가로(w) 세로(h) (전체 이미지 기준, 0과 1 사이값)
  - confidence : object가 있을 확률
    ![o_c](https://user-images.githubusercontent.com/33515697/45637630-f0800900-bae5-11e8-8361-6bda99d049b0.png)
    
- 각 grid cell은 C개의 class를 예측하는 확률값을 예측한다
  ![c_c](https://user-images.githubusercontent.com/33515697/45637618-eeb64580-bae5-11e8-8a4d-dadc440dfcc2.png)

=> 이러한 예측값은 **S X S X (B * 5 + C)** 크기의 tensor로 구성

 </br>

## 네트워크 디자인

![network](https://user-images.githubusercontent.com/33515697/45638600-9fbddf80-bae8-11e8-952f-fc5cc064c3df.png)

*논문은 S=7,B=2,C=20 예제로 하고 있다.

 </br>

**\* 24개의 convolution layers**(+4개의 maxpool) + **2개의 fully connected layers**

- 20개의 convolution layers 

  : 이미지로부터 **feature** 추출

- 4개의 convolution layers + 2개의 fully connected layers 

  : **object classifier** 

- final layer는 S X S X (B * 5 + C)

- final layer가 Bounding box와 class 확률 예측 
 
- final layer는 linear activation 사용, 나머지는 leaky ReLU activation 사용

 </br>

## Loss Function (Sum-Squared Error)

![loss](https://user-images.githubusercontent.com/33515697/45637628-efe77280-bae5-11e8-9241-570d2a4762d2.png)

 </br>

- Localization loss (예측 Boundary box 와 실제 box의 에러)

  ![local_loss](https://user-images.githubusercontent.com/33515697/45637627-efe77280-bae5-11e8-8a56-1b15a0f144ac.png)


  - object 없는 grid cell이 영향력이 크면 안된다

    => ![noobj](https://user-images.githubusercontent.com/33515697/45637985-c8dd7080-bae6-11e8-9a5c-85260ee7420b.PNG)


  - 큰 box의 편차를 작은 box의 편차보다 더 작게 반영해야 한다.

    => w,h 제곱근으로 계산
    
     </br>

- Confidence loss (measure objectness)

  ![confidence_loss](https://user-images.githubusercontent.com/33515697/45637623-ef4edc00-bae5-11e8-9282-3c6ad0affa0d.png)
  
  box안에 object 없을 때,
  
  ![conf_noobj_loss](https://user-images.githubusercontent.com/33515697/45637622-ef4edc00-bae5-11e8-84fa-17b3fc42cb17.png)
  
   </br>

- Classification loss

  ![classification_loss](https://user-images.githubusercontent.com/33515697/45637620-eeb64580-bae5-11e8-9365-80e3b2d36645.png)

 </br>

## Inference

object당 하나의 bounding box만 있어야 한다

=> **Non-max suppression**으로 해결

- Non-max suppression : 최대값만 남기고 나머지 억제

  1. confidence(=box confidence * class prob) 점수로 sort
  
  2. confidence 낮은 것은 0으로 만든다 (e.g. confidence<=0.6 -> set 0) 
  
  3. 가장 큰 값의 confidence 고른다.
  
  4. confidence 순으로 진행, 이전의 bounding box와 비교 순서인 bounding box IOU 비교하여 IOU>=0.5 인 것 억제한다.
  
  5. class 수 만큼 반복


 </br>

#### 참고 문헌 및 사이트

- [논문 pdf](https://pjreddie.com/media/files/papers/yolo.pdf)

- [논문 번역](http://dhhwang89.tistory.com/51)

- https://medium.com/@jonathan_hui/real-time-object-detection-with-yolo-yolov2-28b1b93e2088

- https://medium.com/adventures-with-deep-learning/yolo-v1-part-1-cfb47135f81f

















  
  


