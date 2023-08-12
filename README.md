# PRACTICES OF AUTONOMOUS DRIVING 
## Finding Lane Lines
* 利用OpenCV處理影像，使我們可以在圖片上找出車道線。大致的流程是1.將圖片灰階化2.邊緣偵測3.特徵區間擷取4.找出直線5.合成直線與原圖。
* 老師上課介紹的5種圖形處理技巧
![](https://i.imgur.com/Wo5QwGs.jpg)
![](https://i.imgur.com/D8VgQc5.png)
* 組合這五種技巧，使我們可以在圖片上找出車道線。
## Data
* Github-JoyceYa/CarND-LaneLines-P1-master所提供的test_images。左車道線破碎、左車道線是黃線、車道帶有弧度、右車道線破碎等情況
![](https://i.imgur.com/0EXFy5W.png)
* test_videos， solidWhiteRight.mp4左邊車道線破碎，帶有一些弧度；solidYellowLeft.mp4左邊車道線是黃線，右邊車道線破碎。
![](https://i.imgur.com/LM9ZWUV.png)
* 這段影片是台灣中山高速公路由南至北的全程路景圖，其中包含有直線、彎道、車道變換等等。國道一號 中山高速公路 北向 高雄-基隆 374K-0K 全程 路程景National Highway No. 1(https://www.youtube.com/watch?v=0crwED4yhBA&t=8574s)
{%youtube 0crwED4yhBA %}所有圖片與影片皆取自此部影片
## Step1- Canny
* 由於我們的目的是找出圖片上的車道線，因此先將圖片灰階化，降低複雜度，進一步做高斯模糊，減少圖片的雜訊，降低圖片細節層次，做Canny邊緣偵測運算，找出圖片上的特徵物件。
* 程式碼
![](https://i.imgur.com/Y3vfwrp.png)
* 原始圖片
![](https://i.imgur.com/A0Hij01.jpg)
* 灰階化圖片
![](https://i.imgur.com/DzHiLI4.jpg)
* Gaussian Blur 高斯模糊
![](https://i.imgur.com/ALMaQKE.jpg)
* Canny 邊緣偵測
![](https://i.imgur.com/etiVmiJ.jpg)
* 測試結果
![](https://i.imgur.com/0EXFy5W.png)
![](https://i.imgur.com/KPprmrR.png)

## Step2- Region of interest
* 車道線只會分布在圖像的中下部梯形或三角形的部分，除了車道的區塊，對這個專案而言都是不必要的，因此使用一個遮罩將感興趣外的區域全部都遮掉。
* 程式碼   Trapezoid定義梯形4個頂點，由於圖像下部有時會拍到車子的前蓋、網路上找的圖片右下角有浮水印，為了避免雜訊，將圖片下方一部分也遮掉。
![](https://i.imgur.com/6a91eOp.png)
* 找出有興趣的部分
![](https://i.imgur.com/nqKPrIa.jpg)
* Region of interest
![](https://i.imgur.com/xQANQTG.jpg)


## Step3- Find lanes lines (Image)
* 進行完Step2，我們離目標已經很近了，先用hough transform(霍夫轉換)找出直線；接著判斷線的斜率來區分是左邊車道還是右邊車道(斜率>0:左邊；斜率<0:右邊)；除此之外，由於車道線都是有個弧度，因此設計一個if function來過濾掉斜率小於0.35的線段，這些線段對我們來說是雜訊。最後把線段連起來，合成到原圖上。
* 利用霍夫轉換找到直線(HoughLinesP)
![](https://i.imgur.com/zgnKrI8.png)
image： 必須是二值圖像，推薦使用canny邊緣檢測的結果圖像； 
rho: 線段以像素為單位的距離精度，double類型的
theta： 線段以弧度為單位的角度精度，推薦用pi/180 
threshod: 累加平面的閾值參數，int類型，超過設定閾值才被檢測出線段，值越大，基本上意味著檢出的線段越長，檢出的線段個數越少。
minLineLength：線段以像素為單位的最小長度，根據應用場景設置 
maxLineGap：同一方向上兩條線段判定為一條線段的最大允許間隔（斷裂），超過了設定值，則把兩條線段當成一條線段，值越大，允許線段上的斷裂越大，越有可能檢出潛在的直線段
* 計算找出直線之斜率，做分類若(斜率>0:左邊車道;斜率<0:右邊車道)
![](https://i.imgur.com/UPAV7aA.png)
 "if abs(slope) >0.35: "過濾掉幾乎水平於x軸的線段，避免下一步連直接直線時發生誤判
* 把線連接起來 
![](https://i.imgur.com/C1h3Q05.png)
![](https://i.imgur.com/UfFV16g.jpg)
* 把線加到原圖片上
![](https://i.imgur.com/Vpps9Bx.jpg)
* 測試結果
![](https://i.imgur.com/nznEV2r.png)


## Step4- Find lanes lines (Video)
* 圖片的車道偵測做出來後，將輸入源換成影像，將加上車道線的影像輸出。這邊進行了4個測試來驗證程式在不同情況下的表現。兩個直線測試的影像可以看到在前方暢通的直線車道時，這隻程式可以很精準的判斷出車道線；彎道測試的影像，可以看到雖然在彎道仍可以判斷出車道線，但由於專案做得的是直線偵測，所以在彎道具有弧度的線段不能準確的標示；變換車道的影像，當車子從A車道換到B車道的過程中，這個時間段程式沒辦法辨認出車道線；當轉換到B車道後，車道線可以偵測。因為專案做了興趣區域的提取，將不感興趣的地方用遮罩擋掉了，在車道轉換的過程中，破碎的直線特徵不支持去畫出車道線。
* 程式碼
![](https://i.imgur.com/Qrnn7Je.png)
用cv2讀入影片套用上述function
* [solidWhiteRight.mp4](https://youtu.be/VrqsS01OYG0)
{%youtube VrqsS01OYG0 %}
* [solidYellowLeft.mp4](https://youtu.be/E02ehjjX5k0)
{%youtube E02ehjjX5k0 %}
* [challenge_lane.mp4](https://youtu.be/mWMG-STBgRg)
{%youtube mWMG-STBgRg %}
* [直線車道測試1](https://www.youtube.com/watch?v=_4ffVY6PvpE) 
([原影片](https://youtu.be/0crwED4yhBA?t=8072)2:14:32-2:15:02  )
{%youtube _4ffVY6PvpE %}
* [直線車道測試2](https://www.youtube.com/watch?v=Et2Uri6-dYs) 
([原影片](https://youtu.be/0crwED4yhBA?t=8580)2:23:00-2:23:30 )
{%youtube Et2Uri6-dYs %} 
* [彎道車道測試]( https://www.youtube.com/watch?v=_HJ1MUjegf0) 
([原影片](https://youtu.be/0crwED4yhBA?t=8580)4:13:50-4:14:20 )
{%youtube _HJ1MUjegf0 %} 
* [變換車道測試](https://www.youtube.com/watch?v=ECSGXgUrUFA) 
([原影片](https://youtu.be/0crwED4yhBA?t=8063)2:14:02-2:14:32 )
{%youtube ECSGXgUrUFA %}

## Discussion
* 台灣的高速公路偶爾會有奇怪的水平X軸的線段，例如:
![](https://i.imgur.com/fv3PCw6.jpg)
* 這種情況下，由於水平X軸的線段也在興趣區裡面，形成雜訊判斷車道會失準
![](https://i.imgur.com/aUlNrta.jpg)
![](https://i.imgur.com/wmUyiAs.jpg)
![](https://i.imgur.com/lwAZgIO.jpg)
* 一開始的想法是在興趣區的內部再加一個遮罩，把水平X軸的線過濾掉。
![](https://i.imgur.com/WdPzP1u.jpg)
![](https://i.imgur.com/Tn3oJn9.jpg)
![](https://i.imgur.com/b8TmkF5.jpg)
* 雖然是有修正一些車道偏移但整體準確度依然不行。內部遮罩的位子、大小該如何決定是一個很大的問題。這不是一個好方法。
* 因此放棄這個想法，從另一個角度切入，車道線通常都是有個幅度，因此我令slope=0.35為閥值，過濾掉斜率小於0.35的片段。
![](https://i.imgur.com/LSmhWPu.jpg)
![](https://i.imgur.com/21ckNUs.jpg)
![](https://i.imgur.com/YmRLK6m.jpg)過濾掉斜率小於0.35的片段後可以準確辨識出車道線。
## Authors
[N96094196 張維峻](https://hackmd.io/@po6GeGxZSG-RrxfU2_EF0A/SkvYUnrLd)



