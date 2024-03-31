---
title: 阿里云OSS
date: 2024-3-25 21:05:30
tags: 技术
categories: 
- 工程
- OSS
---







# 基本概念

![image-20240325205911939](C:/Users/夏冬/AppData/Roaming/Typora/typora-user-images/image-20240325205911939.png)













# go操作阿里云



## (1)安装sdk

```bash
package main

import (
	"fmt"
	"github.com/aliyun/aliyun-oss-go-sdk/oss"
)

func main() {
	fmt.Println("OSS Go SDK Version: ", oss.Version)
}

```



## (2)文件上传和下载

```go
package main

import (
	"fmt"
	"github.com/aliyun/aliyun-oss-go-sdk/oss"
	"log"
	"os"
)

//基本配置信息
const (
	OSSBucket          = "bucket-fileserver-store"
	OSSEndpoint        = "oss-cn-beijing.aliyuncs.com"
	OSSAccessKeyID     = ""
	OSSAccessKeySecret = ""
)

const (
	OssPath = "test/"
)

var ossClient *oss.Client

// OssClient 获取oss客户端
func OssClient() (*oss.Client, error) {
	if ossClient != nil {
		return ossClient, nil
	}
	client, err := oss.New(OSSEndpoint, OSSAccessKeyID, OSSAccessKeySecret)
	if err != nil {
		return nil, err
	}
	ossClient = client
	return ossClient, nil
}

// Bucket 获取bucket
func Bucket() (*oss.Bucket, error) {
	client, err := OssClient()
	if err != nil {
		return nil, err
	}
	bucket, err := client.Bucket(OSSBucket)
	if err != nil {
		return nil, err
	}
	return bucket, nil
}

func main() {
	bucket, err := Bucket()
	if err != nil {
		panic(err)
	}
	//创建一个文件流
	fd, err := os.Open("./create.txt")

	err = bucket.PutObject(
		OssPath+"create.txt",
		fd)
	if err != nil {
		panic(err)
	}
	//获取下载链接
	url := DownloadURL(OssPath + "create.txt")
	fmt.Println(url)
}

func UploadObject(objectName, localFileName string) error {
	//获取bucket
	bucket, err := Bucket()
	if err != nil {
		return err
	}
	//打开文件
	fd, err := os.Open(localFileName)
	if err != nil {
		return err
	}
	//上传文件
	err = bucket.PutObject(objectName, fd)
	if err != nil {
		return err
	}
	return nil
}

// DownloadURL 获取文件下载url
func DownloadURL(objectName string) string {
	//获取bucket
	bucket, err := Bucket()
	if err != nil {
		panic(err)
	}
	//获取下载链接
	signedURL, err := bucket.SignURL(objectName, oss.HTTPGet, 3600)
	if err != nil {
		log.Println(err)
		return ""
	}
	return signedURL
}

```

