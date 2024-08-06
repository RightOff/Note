# M1

教程博客：[M1 打印进程树 (pstree) 实验指北 (topdeoo.github.io)](https://topdeoo.github.io/posts/jyyos/jyy-os-m1/)

## pstree.c

```
#include <stdio.h>  //包含操作文件的FILE
#include <assert.h>

#include <string.h>
#include <getopt.h> 
#include <sys/types.h>
#include <dirent.h> //操作目录
#include <malloc.h>
#include <errno.h>
#include <stdlib.h> //以使用 exit 函数
#include <ctype.h>


#define VERSION "VERSION:1.0.0"
#define MAX_PROC 500    //最大可处理进程数

static struct option longopts[] = {
    {"show-pids",0,NULL,'p'},
    {"numeric-sort",0,NULL,'n'},
    {"version",0,NULL,'V'},
    {0,0,0,0}
};
struct process_node{
    pid_t pid;
    pid_t ppid;
    char name[64];

    int cpids[MAX_PROC];
    short cpid_num;
    int vis;    //打印时的访问标志位
};
struct process_node* process_hash_table[MAX_PROC] = {NULL}; //进程信息列表,NULL进行初始化
char opt;   //命令行操作参数
int show_id;    //是否显示id
int is_sort;    //是否排序

int hash(pid_t pid);    //获取可插入的hash值
int get_index(int pid); //根据pid获取哈希表中的下标
struct process_node* get_proc(pid_t pid);   //根据pid获取哈希表中的结点
void buildProcessTree();    //构建进程树
void printProcessTree(struct process_node *head, int depth);    //从某一进程开始打印进程树

void insert_tree(struct process_node *process);  //进程结点插入进程树

int main(int argc, char *argv[]) {
  
    int option_index = -1;
  
    if(argc > 1){
        while((opt = getopt_long(argc,argv,"pnV",longopts,&option_index)) != -1){
            switch(opt){
                case 'p': 
                    //显示pid
                        show_id=1;
                        break;
                case 'n': 
                    //按pid从小到大顺序排列，输出一个进程的直接孩子
                    //实际情况在处理时原本就是升序排列，因此此处不处理
                        is_sort=1;
                        break;
                case 'V': 
                        //版本信息。与系统pstree -V保持一致，输出到标准错误流stderr，然后会像标准输出流stdout一样，打印到控制台
                        //但是./pstree-64 -V > log 将无法写入log，必须用 ./pstree-64 -V 2> log ，因为2>才是stderr的重定向
                        fprintf(stderr, "%s\n",VERSION);  
                        return 0;

                default:
                        //在参数输入错误或无法处理时，将信息写入标准输出
                        fprintf(stderr,"pstree\n");
                        fprintf(stderr,"A convenient tool to show running processes as a tree.\n");
                        fprintf(stderr,"Use -p -n -V to see more\n");
                        return -1;
            }
        }
    }
    assert(!argv[argc]);
  
    //构建进程树
    buildProcessTree();
    //打印进程树
    printProcessTree(get_proc(0), -1);

  
    return 0; 
}

void buildProcessTree(){

    //添加pid=0的结点
    struct process_node *process0 = (struct process_node*)malloc(sizeof(struct process_node));
    strcpy(process0->name, "0");
    process0->pid = 0;
    process0->ppid = 0;

    insert_tree(process0);


    char proc_dir_path[6] = "/proc";
    DIR *proc_dir = opendir(proc_dir_path);   //打开系统中包含全部进程的文件夹
    if (proc_dir == NULL) {
        // 打开目录失败，打印错误信息
        fprintf(stderr, "Error: Could not open directory %s: %s\n", proc_dir_path, strerror(errno));
  
    }
    struct dirent *pro_entry;   //存储文件夹中的条目


    while((pro_entry = readdir(proc_dir))!= NULL){
        //目录条目是数字的就是进程对应的文件夹，数字即进程号
  
        if(isdigit(pro_entry->d_name[0])){
  
            //打开包含当前进程信息的文件
            char proc_info_path[32];
            snprintf(proc_info_path,sizeof(proc_info_path),"%s/%.8s/stat",proc_dir_path, pro_entry->d_name);
            FILE *pro_stat = fopen(proc_info_path,"r");
            assert(pro_stat);
  
            //获取进程信息，加入进程列表
            struct process_node *process = (struct process_node*)malloc(sizeof(struct process_node));
            fscanf(pro_stat,"%d (%[^)]) %*c %d",&process->pid, process->name, &process->ppid);  //(%[^)])是如何匹配字符的
  
            insert_tree(process);
  
            // if(process->ppid != 0){
            //     struct process_node *parent = get_proc(process->ppid);
            //     if(parent == NULL)  
            //         printf("parent为空,process->ppid为：%d,本进程id为：%d\n",process->ppid,process->pid);
            //     if(parent != NULL)
            //         parent->cindex[parent->cpid_num++] = get_index(process->pid);

            // }

            fclose(pro_stat);
        }
    }

    //寻找父进程结点
    for(int i = 1;i<MAX_PROC;++i){
        struct process_node *process = process_hash_table[i];
        if(process != NULL){
            struct process_node *parent = get_proc(process->ppid);
            if(parent != NULL)
                parent->cpids[parent->cpid_num++] = process->pid;
        }

    }
  
    free(pro_entry);
    closedir(proc_dir);
}


int hash(int index){
    return index % MAX_PROC;
}
void insert_tree(struct process_node *process){
    int index = hash(process->pid);
    while(process_hash_table[index] != NULL)
        index = hash(index+1);

    process_hash_table[index] = process;
}

struct process_node* get_proc(pid_t pid){
    int index = hash(pid);
    int start_index = index;

    while(process_hash_table[index] != NULL){   //找不到下标，一般不会存在该情况，为32位系统准备的。为什么？？
        if(process_hash_table[index]->pid == pid){
            return process_hash_table[index];
        }
        index = hash(index+1);
        if(index == start_index)
            break;  //防止死循环
    }

    return NULL;
}

int get_index(int pid){
    int index = hash(pid);
    int start_index = hash(pid);

    while(process_hash_table[index] != NULL){
        if(process_hash_table[index]->pid == pid)  
            return index;   //找到了
  
        index = hash(index+1);
        if(index == start_index)    //防止死循环
            break;
    }
  
    return -1;  //没有找到
}

int compare(const void *a,const void *b){
    return (*(int*)a - *(int*)b);
}

void printProcessTree(struct process_node *head, int depth){
    if(head == NULL || head->vis == 1)  return;
    if(is_sort && head->cpid_num >1){
        qsort(head->cpids,head->cpid_num,sizeof(int), compare);
    }
    head->vis = 1;
    //不打印pid=0的结点
    if(show_id == 1 && head->pid != 0 )
        printf("%*s%s(%d)\n",depth*4,"|",head->name,head->pid);
    else if(show_id != 1 && head->pid != 0)
    {
        printf("%*s%s\n",depth*4,"|",head->name);
    }

    for (int i = 0; i < head->cpid_num; i++){
        printProcessTree(process_hash_table[get_index(head->cpids[i])],depth+1);
    }
}

```

待解决的问题：考虑更贴合实际pstree的输出： 对于叶子节点 /proc/1234/task/ 文件夹下会存在叶子节点开启的子进程( 还会有个该进程的副本文件夹 )文件夹，里面stat文件记录pid和进程名称，且这种进程不会在/proc/ 下出现，pstree中这种进程输出样式为 `[{   ***  }}]`

# L0

# 课程

## 20-动态链接和加载

ELF代码的跳转

### PLT（Procedure Linkage Table）和GOT（Global Offset Table）

[非常详细地解释plt&amp;got (zoepla.github.io)](https://zoepla.github.io/2018/04/%E9%9D%9E%E5%B8%B8%E8%AF%A6%E7%BB%86%E8%A7%A3%E9%87%8Aplt&got/)

PLT条目

```
printf@plt:
    jmp *printf@got(%rip)        # 通过 GOT 跳转
    pushq $relocation_offset     # 将重定位偏移量压栈
    jmp *got_entry_for_dynamic_linker(%rip)  # 跳转到动态链接器，其会将printf实际地址填充到GOT中
```

GOT条目

```
printf@got:
    .quad printf_actual_address  # 初始时会跳转回PLT继续执行，之后会直接跳转到函数位置
```

数据的链接更为麻烦(略)？？

`man ld.so`查看官方文档解释动态链接

## 22-进程的实现

1000个进程都加载一个100MB 的 libbloat.so，通过按需加载、共享内存页、内存映射文件机制实现。

1000个进程都加载一个100GB的 libbloat.so，通过按需加载、共享内存页、内存映射文件、KMS机制实现。

**KSM使用场景示例：**

假设有以下三个进程P1, P2, P3，它们分别加载了一些相同和相似的数据：

* **P1**加载了libbloat.so的一部分内容，同时生成了一些动态数据。
* **P2**加载了相同的libbloat.so内容，同时生成了一些与P1相同的动态数据。
* **P3**加载了libbloat.so的另一部分内容，并生成了与P1, P2相同的动态数据。

在按需加载和内存映射文件的帮助下，P1, P2, P3可以共享libbloat.so的内容，但它们生成的动态数据在物理内存中是独立存在的。

KSM通过扫描这些进程的内存页，发现它们生成的动态数据是相同的，然后将这些内存页合并为一个共享页。这进一步减少了物理内存的使用，提升了内存利用率。
