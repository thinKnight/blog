title: 词法分析器
date: 2014-10-19 19:30:27
tags: Compile
---
一个词法分析器的demo。  
<!-- more -->

## 源码
```C
#include <ctype.h>
#include <stdio.h>
#include <string.h>

int i, j;
int len;
int count;
int start_p;

int analyse(char *tmp, int start_p, FILE *fp_w)
{
    while( isspace(tmp[start_p]) )
        start_p++;

    if( isalpha(tmp[start_p]) )
    {
        count = 0;
        char tmp_str[1000] = {};

        while( isalpha(tmp[start_p]) )
        {
            tmp_str[count] = tmp[start_p]; 
            count++;
            start_p++;
        }
        start_p--;

        if( is_keyword(tmp_str, fp_w) ){
            ;
        }
        else
        {
            printf("<0, %s>\n", tmp_str);
            fprintf(fp_w, "<0, %s>\n", tmp_str);
        }
    }
    else{

        if( isdigit(tmp[start_p]) )
        {
            count = 0;
            char tmp_str[1000] = {};

            while( isdigit(tmp[start_p]) )
            {
                tmp_str[count] = tmp[start_p]; 
                count++;
                start_p++;
            }
            start_p--;
            printf("<1, %s>\n", tmp_str);
            fprintf(fp_w, "<1, %s>\n", tmp_str);
        }
        else{

            if( !isalpha(tmp[start_p]) && !isdigit(tmp[start_p]) )
            {
                count = 0;
                char tmp_str[1000] = {};

                if( !isalpha(tmp[start_p+1]) && !isdigit(tmp[start_p+1]) )
                {
                    tmp_str[count] = tmp[start_p];
                    count++;	start_p++;
                    tmp_str[count] = tmp[start_p];
                    if( is_symbol(tmp_str, fp_w) ){
                        ;
                    }else{
                        count--;    start_p--;
                        tmp_str[1] = '\0';
                        tmp_str[count] = tmp[start_p];
                        is_symbol(tmp_str, fp_w);
                    }

                }else{
                    tmp_str[count] = tmp[start_p]; 
                    is_symbol(tmp_str, fp_w);
                }	
            }

        }
    }

    len = strlen(tmp);
    if( start_p == len )
        return 1;
    else
        analyse(tmp, start_p+1, fp_w);
}


int is_keyword(char *tmp, FILE *fp_w){

    char *keyword[] = {
        "main",   "void",    "int",    "float",  "char",     "double",   "for", 
        "if",     "else",    "while",  "do",     "break",    "switch",   "case",
        "struct", "typedef", "char",   "return", "continue", "sizeof"
    };
    //total number:20
    for (i = 0; i < 20; ++i)
    {
        if( strcmp( tmp, keyword[i]) == 0 )
        {
            printf("<%d, %s>\n", i+2, tmp);
            fprintf(fp_w, "<%d, %s>\n", i+2, tmp);
            return 1;
        }
    }
    return 0;
}

int is_symbol(char *tmp, FILE *fp_w){

    char *symbol[] = {

        "+",  "-",  "*",  "/",  "<",  ">",  "=",  "!",  "&",  "|", 
        "\\", "#",  "\"", "%",  "^",

        "++", "+=", "--", "-=", "*=", "/=", "%=", "&=", "^=", "|=", 
        "<=", ">=", "==", "!=", "&&", "||", ">>", "<<", "?:", "->",

        "{",  "}",  "(",  ")",  "[",  "]",  ",",  ";",  ":",  "."
    };
    //total number:45

    for (i = 0; i < 45; ++i)
        if( strcmp( tmp, symbol[i]) == 0 )
        // printf("Yes! No.%d: %s\n", i, tmp);
        {
            printf("<%d, %s>\n", 22 + i, tmp);
            fprintf(fp_w, "<%d, %s>\n", 22 + i, tmp);
            return 1;
        }

    return 0;
}


int is_blankline(char *tmp){
    int i;
    int len = strlen(tmp);

    for (i = 0; i < len; ++i)
    {
        if( !isspace(tmp[i]) )
            return 0;	
    }
    return 1;

}

int delete_comments(char *tmp)
{
    len = strlen(tmp);
    for(i = 0; i < len; i++)
    {
        if(tmp[i] == '/' && tmp[i+1] == '/')
        {
			
            tmp[i] = '\0';
			
        }
    }
}

int main(int argc, char const *argv[])
{
    char tmp[100] = "";
    char str[100] = "";
    int flag = 0;

    FILE *fp, *fp_tmp;
    fp = fopen("test.c", "r");
    fp_tmp = fopen("tmp.c", "w");
    while(fgets(str, 100, fp) != NULL)
    {
        for(i = 0; i < 100; i++)
        {
            if(str[i] == '/' && str[i+1] == '/')
            {
                str[i] = '\n';
                str[i+1] = '\0';
            }

            if(str[i] == '/' && str[i+1] == '*')
            {
                str[i] = '\0';
                flag = 1;
                fputs(str, fp_tmp);
            }
            if(str[i] == '*' && str[i+1] == '/')
            {
                for (j = 0; j < 100; ++j)
                {
                    str[j] = str[j+i+2];
                }
                str[j] = '\0';
                flag = 0;
            }
        }
        if( flag == 0 )
            fputs(str, fp_tmp);
    }
    fcloseall();

    FILE *fp_new;
    FILE *fp_w;
    fp_new = fopen("tmp.c", "r");
    fp_w   = fopen("Lex_Result.txt", "w");

    while(!feof(fp_new)){
        if(fgets(tmp, 100, fp_new))
            if( is_blankline(tmp) == 0)
                analyse(tmp, 0, fp_w);
    }

    fcloseall();
    remove("tmp.c");
    return 0;
}
```
## 实例
### 目标程序
```C
#include <stdio.h> //This is a simple target code
int main()
{
    printf("TEST BEGUN!");
    int a = 0;
    int b = 1;
    /*
        You can not see me!
    */
    if(a <= b)
        printf("Yes");
 
    return 0;
    //That's it!
}
```  

### 分析结果
```
<33, #>
<0, include>
<26, <>
<0, stdio>
<66, .>
<0, h>
<27, >>
<4, int>
<2, main>
<59, (>
<60, )>
<57, {>
<0, printf>
<59, (>
<34, ">
<0, TEST>
<0, BEGUN>
<29, !>
<34, ">
<60, )>
<64, ;>
<4, int>
<0, a>
<28, =>
<1, 0>
<64, ;>
<4, int>
<0, b>
<28, =>
<1, 1>
<64, ;>
<9, if>
<59, (>
<0, a>
<47, <=>
<0, b>
<60, )>
<0, printf>
<59, (>
<34, ">
<0, Yes>
<34, ">
<60, )>
<64, ;>
<19, return>
<1, 0>
<64, ;>
<58, }>
```
