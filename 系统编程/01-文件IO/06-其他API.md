## 一 文件的随机读/写

API：
```c
int fseek(FILE *stream, long offset, int whence);
功能：移动文件流（文件光标）的读写位置。
参数：
	stream：已经打开的文件指针
	offset：根据whence来移动的位移数（偏移量），可以是正数，也可以负数，如果正数，则相对于whence往右移动，如果是负数，则相对于whence往左移动。如果向前移动的字节数超过了文件开头则出错返回，如果向后移动的字节数超过了文件末尾，再次写入时将增大文件尺寸。
	whence：其取值如下：
		SEEK_SET：从文件开头移动offset个字节
		SEEK_CUR：从当前位置移动offset个字节
		SEEK_END：从文件末尾移动offset个字节
返回值：
	成功：0
	失败：-1

#include <stdio.h>
long ftell(FILE *stream);
功能：获取文件流（文件光标）的读写位置。
参数：
	stream：已经打开的文件指针
返回值：
	成功：当前文件流（文件光标）的读写位置
	失败：-1

#include <stdio.h>
void rewind(FILE *stream);
功能：把文件流（文件光标）的读写位置移动到文件开头。
参数：
	stream：已经打开的文件指针
返回值：
	无返回值
```

示例：
```c
typedef struct Stu{
	char name[50];
	int id;
}Stu;

//假如已经往文件写入3个结构体
//fwrite(s, sizeof(Stu), 3, fp);

Stu s[3];
Stu tmp; 
int ret = 0;

//文件光标读写位置从开头往右移动2个结构体的位置
fseek(fp, 2 * sizeof(Stu), SEEK_SET);

//读第3个结构体
ret = fread(&tmp, sizeof(Stu), 1, fp);
if (ret == 1){
	printf("[tmp]%s, %d\n", tmp.name, tmp.id);
}

//把文件光标移动到文件开头
//fseek(fp, 0, SEEK_SET);
rewind(fp);

ret = fread(s, sizeof(Stu), 3, fp);
printf("ret = %d\n", ret);

int i = 0;
for (i = 0; i < 3; i++){
	printf("s === %s, %d\n", s[i].name, s[i].id);
}
```

## 二 文件的删除与重命名

```c
#include <stdio.h>
int remove(const char *pathname);
功能：
    删除文件
参数：
	pathname：文件名
返回值：
	成功：0
	失败：-1

#include <stdio.h>
int rename(const char *oldpath, const char *newpath);
功能：
    把oldpath的文件名改为newpath
参数：
    oldpath：旧文件名
    newpath：新文件名
返回值：
    成功：0
    失败： - 1
```

## 三 判断文本文件是Linux格式还是Windows格式

```c
#include<stdio.h>

int main(int argc, char **args){
	if (argc < 2)
		return 0;

	FILE *p = fopen(args[1], "rb");
	if (!p)
		return 0;

	char a[1024] = { 0 };
	fgets(a, sizeof(a), p);

	int len = 0;
	while (a[len]){
		if (a[len] == '\n'){
			if (a[len - 1] == '\r'){
				printf("windows file\n");
			} else {
				printf("linux file\n");
			}
		}
		len++;
	}

	fclose(p);

	return 0;
}
```

## 四 获取文件状态

API：
```c
#include <sys/types.h>
#include <sys/stat.h>
int stat(const char *path, struct stat *buf);
功能：
    获取文件状态信息
参数：
    path：文件名
    buf：保存文件信息的结构体
返回值：
    成功：0
    失败-1
```

文件状态：
```c
struct stat {
	dev_t         st_dev;         	// 文件的设备编号
	ino_t         st_ino;           // 节点
	mode_t        st_mode;   		// 文件的类型和存取的权限
	nlink_t       st_nlink;     	// 连到该文件的硬连接数目，刚建立的文件值为1
	uid_t         st_uid;           // 用户ID
	gid_t         st_gid;           // 组ID
	dev_t         st_rdev;      	// (设备类型)若此文件为设备文件，则为其设备编号
	off_t         st_size;          // 文件字节数(文件大小)
	unsigned long st_blksize;   	// 块大小(文件系统的I/O 缓冲区大小)
	unsigned long st_blocks;    	// 块数
	time_t        st_atime;     	// 最后一次访问时间
	time_t        st_mtime;    		// 最后一次修改时间
	time_t        st_ctime;     	// 最后一次改变时间(指属性)
};
```

示例：
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>

int main(int argc, char **args){
	if (argc < 2)
		return 0;

	struct stat st = { 0 };

	stat(args[1], &st);
	int size = st.st_size;//得到结构体中的成员变量
	printf("%d\n", size);
	return 0;
}
```


阻塞和非阻塞是文件本身的属性, 不是read函数的属性.
普通文件：hello.c
默认是非阻塞的
终端设备：如 /dev/tty
默认阻塞
管道和套接字
默认阻塞