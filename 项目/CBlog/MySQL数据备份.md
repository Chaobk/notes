> NBlog部署维护流程记录（持续更新）：[https://blog.csdn.net/qq_43349112/article/details/136129806](https://blog.csdn.net/qq_43349112/article/details/136129806)

为了避免服务器被攻击，给博客添加了一个MySQL数据备份功能。

此功能是配合博客写的，有些方法直接用的已有的，不会再详细展示代码。



备份大致功能步骤如下：

* 使用mysqldump备份数据
* 对备份文件进行压缩
* 压缩文件上传到OSS
* 文件清理
* 邮件通知
* 适应化改造

详细步骤见下文

![CG](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/CG.jpg)



## 0.备份任务主逻辑

目前暂时使用定时任务触发，以下仅为核心代码。

```java
    /**
     * 定时任务，每周一凌晨四点，备份MySQL的数据
     * 备份逻辑：
     * 1.mysql数据备份到文件
     * 2.备份文件压缩
     * 3.压缩文件上传到OSS
     * 4.残留文件清理
     * 5.备份结果的邮件通知 //TODO
     * 6.适应化改造，改成类似NBlog中的定时任务 //TODO
     */
    @Scheduled(cron = "0 0 4 * * 1")
    public void backUpMySQLData() {
        checkDir(backupDir);
        String dateFormat = simpleDateFormat.format(new Date());
        String fileName = String.format("cblog-%s.sql", dateFormat);
        String compressedFileName = fileName + ".zip";
        String dataPath = backupDir + File.separator + fileName;
        String compressedFilePath = backupDir + File.separator + compressedFileName;
        try {
            log.debug("mysql备份开始");
            // 1.mysql数据备份
            backupData(dataPath);
            // 2.文件压缩
            FileUtils.compressFile(dataPath, compressedFilePath);
            // 3.上传到OSS
            String uploadLink = UploadUtils.upload(compressedFilePath);
            log.info("备份文件({})已上传至OSS({})", compressedFilePath, uploadLink);
            // 4.清除残留文件
            FileUtils.delFileByPath(dataPath, compressedFilePath);
        } catch (IOException e) {
            log.error("mysql数据备份失败");
            log.error(e.getMessage());
        }
    }
```





## 1.mysqldump备份

通过`Runtime.getRuntime().exec(xxx)`执行备份命令，保存到指定路径下。

由于我的`MySQL`是`docker`部署，因此使用了`docker exec`命令。

命令的执行结果会比较长，日志级别建议低一些，下面用的`debug`级别。

需要注意`exec(cmds)`的参数格式，写错的话命令不会执行并且很可能不报错，排查了半个下午。

```java
    /**
     * MySQL数据备份
     * @param dataPath 备份文件的保存路径
     * @throws IOException
     */
    private void backupData(String dataPath) throws IOException {
        long start = System.currentTimeMillis();
        String cmd = String.format("docker exec mysql mysqldump -u%s -p%s cblog > %s", dataSourceProperties.getUsername(), dataSourceProperties.getPassword(), dataPath);
        String[] cmds = {"sh", "-c", cmd};
        log.debug("欲执行命令：{}", cmd);
        try (InputStream inputStream = Runtime.getRuntime().exec(cmds).getInputStream();
             InputStreamReader inputStreamReader = new InputStreamReader(inputStream, StandardCharsets.UTF_8);
             BufferedReader bufferedReader = new BufferedReader(inputStreamReader);) {
            String line = bufferedReader.readLine();
            while (line != null) {
                log.debug(line);
                line = bufferedReader.readLine();
            }
        }
        long end = System.currentTimeMillis();
        log.info("mysql备份命令执行成功,耗时：{}ms", end - start);
    }
```



## 2.备份文件压缩

压缩成zip格式，核心代码如下：

```java
     * 压缩文件到指定路径
     * @param oriFilePath 要压缩的原文件路径
     * @param compressedFilePath 压缩后的文件存放路径
     * @throws IOException IO异常，不在catch模块捕捉，交给调用方自行处理
     */
    public static void compressFile(String oriFilePath, String compressedFilePath) throws IOException {
        File file = new File(oriFilePath);
        File zipFile = new File(compressedFilePath);
        try (FileInputStream fileInputStream = new FileInputStream(file);
             ZipOutputStream zipOutputStream = new ZipOutputStream(Files.newOutputStream(zipFile.toPath()))){
            zipOutputStream.putNextEntry(new ZipEntry(file.getName()));
            int temp = 0;
            while ((temp = fileInputStream.read()) != -1) {
                zipOutputStream.write(temp);
            }
        }
        log.info("文件压缩完成");
    }
```



## 3.上传至OSS

核心代码如下

```java
    public String upload(String filepath) throws IOException {
        File file = new File(filepath);
        String uploadName = aliyunProperties.getBackupPath() + "/" + file.getName();
        PutObjectRequest putObjectRequest = new PutObjectRequest(aliyunProperties.getBucketName(), uploadName, file);
        return uploadByOSS(putObjectRequest, uploadName);
    }

    private String uploadByOSS(PutObjectRequest putObjectRequest, String uploadName) throws IOException {
        try {
            ossClient.putObject(putObjectRequest);
            return String.format("https://%s.%s/%s", aliyunProperties.getBucketName(), aliyunProperties.getEndpoint(), uploadName);
        } catch (Exception e) {
            throw new RuntimeException("阿里云OSS上传失败");
        }
    }
```



## 4.清除残留文件

上传后，备份文件和压缩文件已经无用，删除掉即可：

```java
    public static void delFileByPath(String... paths) {
        if (paths == null) {
            return;
        }
        for (String path : paths) {
            File file = new File(path);
            del(file);
        }
    }

    private static void del(File file) {
        String filePath = file.getAbsolutePath();
        file.delete();
        log.info("{}文件已清除", filePath);
    }
```



## 5.邮件通知结果

已经有了`MailUtils`工具类，可以直接发送结果
```java
// 5.备份结果的邮件通知
mailUtils.sendSimpleMailToMyself("MySQL数据备份", String.format("备份数据链接为(若为null说明备份失败，请检查日志)：%s ", uploadLink));
```

## 6.适应化改造

直接在后台管理的定时任务新增任务即可，注意Cron表达式

![image-20240331183300886](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240331183300886.png)



## X、测试

确定定时任务触发后，`OSS`能看到文件即可

![image-20240317171232043](https://chaobk-img-repo.oss-cn-shanghai.aliyuncs.com/note-md/image-20240317171232043.png)