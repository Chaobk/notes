## Midjourney常用命令

1./settings

| key                 | desc                                               |
| ------------------- | -------------------------------------------------- |
| stylize             | 风格化                                             |
| public mode         | 公共模式                                           |
| remix mode          | 修改模式，生成图片后，可以再次点击图片，修改prompt |
| higt variation mode | 高差异模式，生成的四张图片会有更大的不同           |
| turbo mode          | 急速模式，消耗是快速模式的四倍                     |
|                     |                                                    |



![image-20240519191834285](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240519191834285.png)



/imagine 生图



```shell
/show job-id
```

可以重新拿到图片的任务



```shell
/describe image/link
```

获取上传图片的关键词



```shell
/shorten	
```

精简关键词



## 后缀

| param   | desc                                                         | 示例                        |
| ------- | ------------------------------------------------------------ | --------------------------- |
| --ar    | 宽高比设置                                                   |                             |
| --c     | 多样性设置，数值0-100，默认值0，值越大，四张图的差异越大     |                             |
| --s     | 风格化设置，数值0-1000，默认值100。值越小，越接近关键词效果；值越大，发挥空间越大，更容易脱离关键词效果 |                             |
| --q     | 渲染质量设置。数值 0.25、0.5、1，默认值1                     |                             |
| --stop  | 渲染停止设置。数值10-100，默认值100。为了避免midjourney理解错prompt含义，可以先用这个生成一部分看看效果 |                             |
| --iw    | 上传图片的权重设置。数值0-2，默认值1。值越高，和上传的图片越相似 |                             |
| --seed  | 种子值。4宫格的种子值。                                      |                             |
| --no    | 负面描述词。描述不希望画面出现的内容                         |                             |
| --tile  | 无缝拼接图案。常用于纹理制作                                 |                             |
| --r     | 重复生图。数值范围1-40                                       |                             |
| --niji  | 版本切换。                                                   |                             |
| --video | 记录生图视频                                                 | /imagine a cute dog --video |



## 合格的Prompt提示词

* 优化语句
  * 尽量避免口语化语言
  * 避免使用介词
* 不要冗余
  * 避免使用同质化词
  * 减少权重0的词（可以使用shorten检测）
* 长度合理
  * 简单、明了、直接
  * 尽可能简短
  * 词越少，权重越大
  * 80单词内
* 结构清晰

![image-20240519230845735](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240519230845735.png)

![image-20240519230954232](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240519230954232.png)



## 关键词的化合作用

midjourney中英双语图文词典

Takram Making