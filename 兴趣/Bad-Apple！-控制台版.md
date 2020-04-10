成果视频：[https://www.bilibili.com/video/av96522403/](https://www.bilibili.com/video/av96522403/)
代码：[https://github.com/LPJworkroom/Bad_Apple_in_text/tree/master](https://github.com/LPJworkroom/Bad_Apple_in_text/tree/master)

Bad Apple只有黑白两色，很适合用像素块在控制台重现。
基本思想是，首先得到视频的每一帧，将每一帧转换成方块字符组成的“字符画”，最后在控制台中打印出这些字符画，实现视频的重现。
##视频转成图片
用网上能查到的现成代码，将MP4转为jpg图片。
```
captured: VideoCapture = cv.VideoCapture('bad_apple.mp4')
success, image = captured.read()
count = 0
while success:
    cv.imwrite('frame%d.jpg' % count, image)
    success, image = captured.read()
    count += 1
```
##图片转成字符画
图片转成字符其实就是将一大块像素转成黑色或白色。
想象一个10 * 10像素的小窗口，在图片上从左至右从上至下的滑动。每次滑动向右10像素，向下时也一样。每次滑动统计窗口内10 * 10=100个像素的颜色。黑色是RGB0,0,0，白色是255,255,255，我们将127,127,127作为黑白分界线，如果这100个像素的颜色“偏浅”，也就是100个像素的RGB之和大于100 * (255 * 3/2)，则视为偏白，将转换为一个“█”字符，否则转换为一个“ ”（空格）字符。
```
target = 0  #黑白分界数
        colorSum = 0 
        #  cellHeight cellWidth 即为小窗口的高和宽
        for i in range(row * self.cellHeight, min(row * self.cellHeight + self.cellHeight, imageHeight)):
            for j in range(col * self.cellWidth, min(col * self.cellWidth + self.cellWidth, imageWidth)):
                    target += 255 * 3 / 2
                    colorSum += sum(image_arr[i][j])  #像素和
        return colorSum > target 
```
##播放
播放的问题不大，设定好帧率也就得到了两张图之间间隔时间，按照间隔时间播放即可。
需要注意的是，有时控制台输出一张图片的时间会超过间隔时间，这将造成播放越来越慢。我的解决方法是记录当前时间和当前第几帧，在输出每张图片前计算已经经过的时间；再计算理论上应当经过的时间，即当前帧数/帧率。如果理论时间超过经过时间，即播放“太慢”了，就跳过这一帧。
