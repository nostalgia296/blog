---
title: '在C中使用libcurl进行网络请求'
published: 2025-11-28
description: ''
image: ''
tags: [C]
category: '测试文章'
draft: false 
lang: ''
---

# C语言中使用libcurl进行网络请求

## 简介

libcurl是一个功能强大的开源网络传输库，支持多种协议（HTTP、HTTPS、FTP等），是C语言中进行网络编程的首选工具之一。本文将详细介绍如何在C语言项目中使用libcurl进行各种网络请求。

## 1. 安装libcurl

### Ubuntu/Debian系统
```bash
sudo apt-get install libcurl4-openssl-dev
```

### CentOS/RHEL系统
```bash
sudo yum install libcurl-devel
# 或者对于较新版本
sudo dnf install libcurl-devel
```

### macOS系统
```bash
brew install curl
```

### Windows系统
从[libcurl官网](https://curl.se/windows/)下载预编译的库文件，或者使用vcpkg：
```bash
vcpkg install curl
```

## 2. 基本编译

编译时需要链接libcurl库：
```bash
gcc -o program program.c -lcurl
```

## 3. 简单的HTTP GET请求

```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    // 初始化libcurl
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        // 设置请求URL
        curl_easy_setopt(curl, CURLOPT_URL, "https://api.github.com/users/torvalds");
        
        // 执行请求
        res = curl_easy_perform(curl);
        
        // 检查错误
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        // 清理
        curl_easy_cleanup(curl);
    }
    
    // 全局清理
    curl_global_cleanup();
    return 0;
}
```

## 4. 处理响应数据

上面的例子只是将响应输出到stdout。要处理响应数据，我们需要使用写回调函数：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>

// 结构体用于存储响应数据
typedef struct {
    char *data;
    size_t size;
} response_data_t;

// 写回调函数
static size_t write_callback(void *contents, size_t size, size_t nmemb, void *userp) {
    size_t realsize = size * nmemb;
    response_data_t *mem = (response_data_t *)userp;
    
    // 重新分配内存
    char *ptr = realloc(mem->data, mem->size + realsize + 1);
    if(ptr == NULL) {
        printf("not enough memory (realloc returned NULL)\n");
        return 0;
    }
    
    mem->data = ptr;
    memcpy(&(mem->data[mem->size]), contents, realsize);
    mem->size += realsize;
    mem->data[mem->size] = 0;
    
    return realsize;
}

int main(void) {
    CURL *curl;
    CURLcode res;
    response_data_t response = {0};
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        curl_easy_setopt(curl, CURLOPT_URL, "https://jsonplaceholder.typicode.com/posts/1");
        
        // 设置写回调函数
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, (void *)&response);
        
        // 执行请求
        res = curl_easy_perform(curl);
        
        if(res == CURLE_OK) {
            printf("Response received:\n%s\n", response.data);
            printf("Response size: %zu bytes\n", response.size);
        } else {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        curl_easy_cleanup(curl);
    }
    
    // 释放响应数据内存
    if(response.data) {
        free(response.data);
    }
    
    curl_global_cleanup();
    return 0;
}
```

## 5. HTTP POST请求

### 5.1 发送表单数据

```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        // 设置POST URL
        curl_easy_setopt(curl, CURLOPT_URL, "https://httpbin.org/post");
        
        // 设置为POST请求
        curl_easy_setopt(curl, CURLOPT_POST, 1L);
        
        // 设置POST数据
        const char *post_data = "name=John&age=30&city=NewYork";
        curl_easy_setopt(curl, CURLOPT_POSTFIELDS, post_data);
        
        // 执行请求
        res = curl_easy_perform(curl);
        
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        curl_easy_cleanup(curl);
    }
    
    curl_global_cleanup();
    return 0;
}
```

### 5.2 发送JSON数据

```c
#include <stdio.h>
#include <string.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        // JSON数据
        const char *json_data = "{\"name\":\"Alice\",\"age\":25,\"email\":\"alice@example.com\"}";
        
        // 设置URL
        curl_easy_setopt(curl, CURLOPT_URL, "https://httpbin.org/post");
        
        // 设置为POST请求
        curl_easy_setopt(curl, CURLOPT_POST, 1L);
        
        // 设置POST数据
        curl_easy_setopt(curl, CURLOPT_POSTFIELDS, json_data);
        
        // 设置Content-Type头
        struct curl_slist *headers = NULL;
        headers = curl_slist_append(headers, "Content-Type: application/json");
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
        
        // 执行请求
        res = curl_easy_perform(curl);
        
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        // 清理
        curl_slist_free_all(headers);
        curl_easy_cleanup(curl);
    }
    
    curl_global_cleanup();
    return 0;
}
```

## 6. 设置请求头

```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        // 设置请求头
        struct curl_slist *headers = NULL;
        headers = curl_slist_append(headers, "User-Agent: MyCApp/1.0");
        headers = curl_slist_append(headers, "Accept: application/json");
        headers = curl_slist_append(headers, "Authorization: Bearer your_token_here");
        
        curl_easy_setopt(curl, CURLOPT_URL, "https://httpbin.org/get");
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
        
        res = curl_easy_perform(curl);
        
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        curl_slist_free_all(headers);
        curl_easy_cleanup(curl);
    }
    
    curl_global_cleanup();
    return 0;
}
```

## 7. 超时设置

```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        curl_easy_setopt(curl, CURLOPT_URL, "https://httpbin.org/delay/5");
        
        // 设置连接超时（秒）
        curl_easy_setopt(curl, CURLOPT_CONNECTTIMEOUT, 10L);
        
        // 设置总超时时间（秒）
        curl_easy_setopt(curl, CURLOPT_TIMEOUT, 15L);
        
        // 设置低速度时间（如果传输速度低于此值超过指定时间，则中止）
        curl_easy_setopt(curl, CURLOPT_LOW_SPEED_TIME, 20L);
        curl_easy_setopt(curl, CURLOPT_LOW_SPEED_LIMIT, 1L);
        
        res = curl_easy_perform(curl);
        
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        curl_easy_cleanup(curl);
    }
    
    curl_global_cleanup();
    return 0;
}
```

## 8. HTTPS和SSL验证

```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        curl_easy_setopt(curl, CURLOPT_URL, "https://api.github.com/users/torvalds");
        
        // 禁用SSL证书验证（仅用于测试，生产环境不建议）
        // curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0L);
        // curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0L);
        
        // 设置CA证书路径
        curl_easy_setopt(curl, CURLOPT_CAINFO, "/etc/ssl/certs/ca-certificates.crt");
        
        res = curl_easy_perform(curl);
        
        if(res != CURLE_OK) {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        curl_easy_cleanup(curl);
    }
    
    curl_global_cleanup();
    return 0;
}
```

## 9. 获取响应状态码和头信息

```c
#include <stdio.h>
#include <curl/curl.h>

int main(void) {
    CURL *curl;
    CURLcode res;
    long response_code;
    
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
    
    if(curl) {
        curl_easy_setopt(curl, CURLOPT_URL, "https://httpbin.org/status/404");
        
        // 启用头部信息
        curl_easy_setopt(curl, CURLOPT_HEADER, 1L);
        
        res = curl_easy_perform(curl);
        
        if(res == CURLE_OK) {
            // 获取响应状态码
            curl_easy_getinfo(curl, CURLINFO_RESPONSE_CODE, &response_code);
            printf("Response code: %ld\n", response_code);
            
            // 获取内容长度
            double content_length;
            curl_easy_getinfo(curl, CURLINFO_CONTENT_LENGTH_DOWNLOAD, &content_length);
            printf("Content length: %.0f bytes\n", content_length);
            
            // 获取总时间
            double total_time;
            curl_easy_getinfo(curl, CURLINFO_TOTAL_TIME, &total_time);
            printf("Total time: %.3f seconds\n", total_time);
        } else {
            fprintf(stderr, "curl_easy_perform() failed: %s\n", 
                    curl_easy_strerror(res));
        }
        
        curl_easy_cleanup(curl);
    }
    
    curl_global_cleanup();
    return 0;
}
```

## 10. 完整的封装示例

下面是一个完整的libcurl封装类，提供了常用的网络请求功能：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>

typedef struct {
    char *data;
    size_t size;
} http_response_t;

static size_t write_callback(void *contents, size_t size, size_t nmemb, void *userp) {
    size_t realsize = size * nmemb;
    http_response_t *mem = (http_response_t *)userp;
    
    char *ptr = realloc(mem->data, mem->size + realsize + 1);
    if(ptr == NULL) {
        return 0;
    }
    
    mem->data = ptr;
    memcpy(&(mem->data[mem->size]), contents, realsize);
    mem->size += realsize;
    mem->data[mem->size] = 0;
    
    return realsize;
}

// 初始化HTTP客户端
CURL* http_client_init(void) {
    curl_global_init(CURL_GLOBAL_DEFAULT);
    return curl_easy_init();
}

// 清理HTTP客户端
void http_client_cleanup(CURL *curl) {
    if(curl) {
        curl_easy_cleanup(curl);
    }
    curl_global_cleanup();
}

// 设置通用选项
void http_client_set_common_options(CURL *curl, const char *url, long timeout) {
    curl_easy_setopt(curl, CURLOPT_URL, url);
    curl_easy_setopt(curl, CURLOPT_TIMEOUT, timeout);
    curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L);
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
}

// GET请求
http_response_t* http_get(CURL *curl, const char *url, struct curl_slist *headers) {
    http_response_t *response = malloc(sizeof(http_response_t));
    response->data = malloc(1);
    response->size = 0;
    response->data[0] = '\0';
    
    http_client_set_common_options(curl, url, 30L);
    curl_easy_setopt(curl, CURLOPT_HTTPGET, 1L);
    if(headers) {
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
    }
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, (void *)response);
    
    CURLcode res = curl_easy_perform(curl);
    if(res != CURLE_OK) {
        fprintf(stderr, "GET request failed: %s\n", curl_easy_strerror(res));
        free(response->data);
        free(response);
        return NULL;
    }
    
    return response;
}

// POST请求
http_response_t* http_post(CURL *curl, const char *url, const char *data, 
                          struct curl_slist *headers) {
    http_response_t *response = malloc(sizeof(http_response_t));
    response->data = malloc(1);
    response->size = 0;
    response->data[0] = '\0';
    
    http_client_set_common_options(curl, url, 30L);
    curl_easy_setopt(curl, CURLOPT_POST, 1L);
    if(data) {
        curl_easy_setopt(curl, CURLOPT_POSTFIELDS, data);
    }
    if(headers) {
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
    }
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, (void *)response);
    
    CURLcode res = curl_easy_perform(curl);
    if(res != CURLE_OK) {
        fprintf(stderr, "POST request failed: %s\n", curl_easy_strerror(res));
        free(response->data);
        free(response);
        return NULL;
    }
    
    return response;
}

// 释放响应内存
void http_response_free(http_response_t *response) {
    if(response) {
        if(response->data) {
            free(response->data);
        }
        free(response);
    }
}

int main(void) {
    CURL *curl = http_client_init();
    if(!curl) {
        fprintf(stderr, "Failed to initialize HTTP client\n");
        return 1;
    }
    
    // 设置请求头
    struct curl_slist *headers = NULL;
    headers = curl_slist_append(headers, "Content-Type: application/json");
    headers = curl_slist_append(headers, "User-Agent: CHttpClient/1.0");
    
    // GET请求示例
    printf("=== GET Request ===\n");
    http_response_t *get_response = http_get(curl, "https://httpbin.org/get", headers);
    if(get_response) {
        printf("Response:\n%s\n", get_response->data);
        printf("Size: %zu bytes\n", get_response->size);
        http_response_free(get_response);
    }
    
    // POST请求示例
    printf("\n=== POST Request ===\n");
    const char *post_data = "{\"name\":\"John\",\"age\":30}";
    http_response_t *post_response = http_post(curl, "https://httpbin.org/post", 
                                              post_data, headers);
    if(post_response) {
        printf("Response:\n%s\n", post_response->data);
        printf("Size: %zu bytes\n", post_response->size);
        http_response_free(post_response);
    }
    
    // 清理
    curl_slist_free_all(headers);
    http_client_cleanup(curl);
    
    return 0;
}
```

## 11. 常见问题和解决方案

### 11.1 内存泄漏
确保每次调用`malloc`或`realloc`后都有对应的`free`调用。

### 11.2 线程安全
libcurl在多线程环境中使用时需要注意：
- 不要在多个线程中共享同一个CURL句柄
- 使用`curl_global_init()`和`curl_global_cleanup()`时要小心

### 11.3 错误处理
始终检查`curl_easy_perform()`的返回值，并使用`curl_easy_strerror()`获取错误描述。