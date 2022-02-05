# CS:APP Proxy Lab

### Preface

A Web proxy is a program that acts as a middleman between a Web browser and an end server. Instead of contacting the end server directly to get a Web page, the browser contacts the proxy, which forwards the request on to the end server. When the end server replies to the proxy, the proxy sends the reply on to the browser.



### Part I: Implementing a sequential web proxy

```c
#include <stdio.h>
#include "csapp.h"

/* Recommended max cache and object sizes */
#define MAX_CACHE_SIZE 1049000
#define MAX_OBJECT_SIZE 102400

/* You won't lose style points for including this long line in your code */
static const char *user_agent_hdr = "User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:10.0.3) Gecko/20120305 Firefox/10.0.3\r\n";
static const char *connection_hdr = "Connection: close\r\n";
static const char *proxy_connection_hdr = "Proxy-Connection: close\r\n";
static const char *endof_hdr = "\r\n";
static const char *user_agent_key = "User-Agent";
static const char *connection_key = "Connection";
static const char *proxy_connection_key = "Proxy-Connection";
static const char *host_key = "Host";
static const char *host_hdr_format = "GET %s HTTP/1.0\r\n";
static const char *requestlint_hdr_format = "GET %s HTTP/1.0\r\n";

void doit(int fd);
void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char *longmsg);
int connect_endserver(char *hostname, int port, char *http_header);
void build_http_header(char *http_header, char *hostname, char *path, int port, rio_t *client_rio);
void parse_uri(char *uri, char *hostname, char *path, int *port);

int main(int argc, char** argv)
{
    int listenfd, connfd;
    char hostname[MAXLINE], port[MAXLINE];
    socklen_t clientlen;
    struct sockaddr_storage clientaddr;

    if (argc != 2)
    {
        fprintf(stderr, "usage: %s <port>\n", argv[0]);
        exit(1);
    }

    listenfd = Open_listenfd(argv[1]);
    while (1)
    {
        clientlen = sizeof(clientaddr);
        connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen);
        Getnameinfo((SA *)&clientaddr, clientlen, hostname, MAXLINE, port, MAXLINE, 0);
        printf("Accepted connection from (%s, %s).\n", hostname, port);
        doit(connfd);
        Close(connfd);
    }

    return 0;
}

void doit(int fd)
{
    int endserver_fd;

    char buf[MAXLINE], method[MAXLINE], uri[MAXLINE], version[MAXLINE];
    char endserver_http_header[MAXLINE];
    char hostname[MAXLINE], path[MAXLINE];
    int port;

    rio_t client_rio, server_rio;

    Rio_readinitb(&client_rio, fd);
    Rio_readlineb(&client_rio, buf, MAXLINE);
    sscanf(buf, "%s %s %s", method, uri, version);

    if (strcasecmp(method, "GET"))
    {
        clienterror(fd, method, "501", "Not Implemented", "Tiny does not implement this method");
        return;
    }

    parse_uri(uri, hostname, path, &port);

    build_http_header(endserver_http_header, hostname, path, port, &client_rio);

    endserver_fd = connect_endserver(hostname, port, endserver_http_header);
    Rio_readinitb(&server_rio, endserver_fd);
    Rio_writen(endserver_fd, endserver_http_header, strlen(endserver_http_header));

    size_t n;
    while ((n = Rio_readlineb(&server_rio, buf, MAXLINE)) != 0)
    {
        Rio_writen(fd, buf, n);
    }

    Close(endserver_fd);
}

void clienterror(int fd, char *cause, char *errnum, char *shortmsg, char *longmsg)
{
    char buf[MAXLINE];

    sprintf(buf, "HTTP/1.0 %s %s\r\n", errnum, shortmsg);
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "Content-type: text/html\r\n\r\n");
    Rio_writen(fd, buf, strlen(buf));

    sprintf(buf, "<html><title>Tiny Error</title>");
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "<body bgcolor=""ffffff"">\r\n");
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "%s: %s\r\n", errnum, shortmsg);
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "<p>%s: %s\r\n", longmsg, cause);
    Rio_writen(fd, buf, strlen(buf));
    sprintf(buf, "<hr><em>The Tiny Web server</em>\r\n");
    Rio_writen(fd, buf, strlen(buf));
}

int connect_endserver(char *hostname, int port, char *http_header)
{
    char portstr[100];
    sprintf(portstr, "%d", port);
    return Open_clientfd(hostname, portstr);
}

void build_http_header(char *http_header, char *hostname, char *path, int port, rio_t *client_rio)
{
    char buf[MAXLINE], request_hdr[MAXLINE], other_hdr[MAXLINE], host_hdr[MAXLINE];
    
    sprintf(request_hdr, requestlint_hdr_format, path);
    
    while (Rio_readlineb(client_rio, buf, MAXLINE) > 0)
    {
        if (strcmp(buf, endof_hdr) == 0) break;

        if (!strncasecmp(buf, host_key, strlen(host_key)))
        {
            strcpy(host_hdr, buf);
            continue;
        }

        if (!strncasecmp(buf, connection_key, strlen(connection_key))
                && !strncasecmp(buf, proxy_connection_key, strlen(proxy_connection_key))
                && !strncasecmp(buf, user_agent_key, strlen(user_agent_key)))
        {
            strcat(other_hdr, buf);
        }
    }

    if (strlen(host_hdr) == 0)
    {
        sprintf(host_hdr, host_hdr_format, hostname);
    }

    sprintf(http_header, "%s%s%s%s%s%s%s",
            request_hdr,
            host_hdr,
            connection_hdr,
            proxy_connection_hdr,
            user_agent_hdr,
            other_hdr,
            endof_hdr);

    return;
}

void parse_uri(char *uri, char *hostname, char *path, int *port)
{
    *port = 80;

    char *host_pos = strstr(uri, "//");
    if (host_pos != NULL)
    {
        host_pos = host_pos + 2;
    }
    else
    {
        host_pos = uri;
    }

    char *path_pos = strstr(host_pos, ":");
    if (path_pos != NULL)
    {
        *path_pos = '\0';
        sscanf(host_pos, "%s", hostname);
        sscanf(path_pos + 1, "%d%s", port, path);
    }
    else
    {
        path_pos = strstr(host_pos, "/");
        if (path_pos != NULL)
        {
            *path_pos = '\0';
            sscanf(host_pos, "%s", hostname);
            *path_pos = '/';
            sscanf(path_pos, "%s", path);
        }
        else
        {
            sscanf(host_pos, "%s", hostname);
        }
    }

    return;
}
```



###  Part II: Dealing with multiple concurrent requests

```c
/*
 * main - modified
 */
int main(int argc, char** argv)
{
    int *connfd;
    ...
    while (1)
    {
        clientlen = sizeof(clientaddr);
        connfd = Malloc(sizeof(int));
        *connfd = Accept(listenfd, (SA *)&clientaddr, &clientlen);
        Getnameinfo((SA *)&clientaddr, clientlen, hostname, MAXLINE, port, MAXLINE, 0);
        printf("Accepted connection from (%s, %s).\n", hostname, port);
        Pthread_create(&tid, NULL, thread, connfd);
    }

    return 0;
}

/*
 * thread - added
 */
void *thread(void *vargp)
{
    int connfd = *((int *)vargp);
    Pthread_detach(pthread_self());
    Free(vargp);
    doit(connfd);
    Close(connfd);
    return NULL;
}
```



###  Part III: Caching web objects

```c
int main(int argc, char** argv)
{
    ...
    cache_init();
    ...
}

void doit(int fd)
{
    ...

    char uri_store[MAXLINE];
    strcpy(uri_store, uri);
    int index;
    if ((index = cache_find(uri_store)) != -1)
    {
        read_previous(index);
        Rio_writen(fd, cache[index].obj, strlen(cache[index].obj));
        read_after(index);
        cache_lru(index);
        return;
    }
    
    ...

    char obj_store[MAX_OBJECT_SIZE];
    size_t obj_size = 0, n;
    while ((n = Rio_readlineb(&server_rio, buf, MAXLINE)) != 0)
    {
        obj_size += n;
        if (obj_size < MAX_OBJECT_SIZE)
        {
            strcat(obj_store, buf);
        }
        Rio_writen(fd, buf, n);
    }

    Close(endserver_fd);

    if (obj_size < MAX_OBJECT_SIZE)
    {
        cache_store(uri_store, obj_store);
    }
}

void cache_init()
{
    for (int i = 0; i < 10; i++)
    {
        cache[i].lru = 0;
        cache[i].vld = 0;
        cache[i].reader_cnt = 0;
        cache[i].writer_cnt = 0;
        Sem_init(&cache[i].rc_mutex, 0, 1);
        Sem_init(&cache[i].wc_mutex, 0, 1);
        Sem_init(&cache[i].cache_mutex, 0, 1);
        Sem_init(&cache[i].cache_queue, 0, 1);
    }
}

int cache_find(char *url)
{
    for (int i = 0; i < 10; ++i)
    {
        read_previous(i);
        if ((cache[i].vld == 1) && (strcmp(cache[i].uri, url) == 0))
        {
            read_after(i);
            return i;
        }
        read_after(i);
    }

    return -1;
}

int cache_eviction()
{
    int maxValue = 0;
    int maxIndex = 0;
    for (int i = 0; i < 10; i++)
    {
        read_previous(i);
        if (cache[i].vld == 0)
        {
            maxValue = 0xffffffff;
            maxIndex = i;
            read_after(i);
            break;
        }
        if (maxValue < cache[i].lru)
        {
            maxValue = cache[i].lru;
            maxIndex = i;
            read_after(i);
            continue;
        }
        read_after(i);
    }

    return maxIndex;
}

void cache_store(char *uri, char *buf)
{
    int index = cache_eviction();

    write_previous(index);
    strcpy(cache[index].obj, buf);
    strcpy(cache[index].uri, uri);
    cache[index].vld = 1;
    write_after(index);

    cache_lru(index);
}

void cache_lru(int index)
{
    write_previous(index);
    cache[index].lru = 0;
    write_after(index);

    for (int i = 0; i < 10; ++i)
    {
        if (i != index)
        {
            write_previous(i);
            if (cache[i].vld == 1)
            {
                ++cache[i].lru;
            }
            write_after(i);
        }
    }
}

void read_previous(int i)
{
    P(&cache[i].cache_queue);
    P(&cache[i].rc_mutex);
    cache[i].reader_cnt++;
    if (cache[i].reader_cnt == 1) P(&cache[i].cache_mutex);
    V(&cache[i].rc_mutex);
    V(&cache[i].cache_queue);
}

void read_after(int i)
{
    P(&cache[i].rc_mutex);
    cache[i].reader_cnt--;
    if (cache[i].reader_cnt == 0) V(&cache[i].cache_mutex);
    V(&cache[i].rc_mutex);
}

void write_previous(int i)
{
    P(&cache[i].wc_mutex);
    cache[i].writer_cnt++;
    if (cache[i].writer_cnt == 1) P(&cache[i].cache_queue);
    V(&cache[i].wc_mutex);
    P(&cache[i].cache_mutex);
}

void write_after(int i)
{
    V(&cache[i].cache_mutex);
    P(&cache[i].wc_mutex);
    cache[i].writer_cnt--;
    if (cache[i].writer_cnt == 0) V(&cache[i].cache_queue);
    V(&cache[i].wc_mutex);
}
```



### Reference

[proxylab.pdf (cmu.edu)](http://csapp.cs.cmu.edu/3e/proxylab.pdf)