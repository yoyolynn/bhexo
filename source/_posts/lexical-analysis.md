---
title: 编译原理——词法分析————xyiyy
---

这部分的实验主要是实现词法分析的功能

#### 方法一
比较简单的方法是通过每次枚举所有的关键字，然后不断暴力匹配，复杂度平方

#### 方法二
与方法一类似的是使用正则表达式，相对来说，略好一些，不过，这种方法也需要不断去尝试匹配，复杂度还是平方级的
ps:偷偷附上队友用正则实现的代码[他看不到看不到](http://paste.ubuntu.com/15941872/)
我觉得他写的很好

#### 方法三
我设计了一种复杂度为线性的方法。首先，词法分析的这个问题可以理解为              一个字符串模式匹配的问题,而且很明显这是一个***多模式串匹配***的问题，所以这个地方不能用大家以往熟知的kmp来解决这个问题。当然这个问题和模式匹配还有一定的区别，所以这里我们主要采用了***字典树***这一数据结构来处理问题。
大致的思想可以理解为对于所有的模式串建立一棵字典树（每个模式串的结尾都会打上一个标记tail），然后每次对于目标串不断往下匹配直到不能匹配。在不匹配的时候，可能有三种情况：
1. 这部分的字符是一个之前没有出现过的标识符，那么不断贪心取，取到不能取之后将这段字符标记为标识符插入到字典树中。
2. 这部分的起始字符是数字，那么由于标识符一定是以字母开头的，则可以不断贪心往后取，直到不是数字。
3. 这部分的字符是空格，制表符

主要的代码部分都在字典树的query函数中
```cpp
//#####################
//Author:fraud
//Blog: http://www.cnblogs.com/fraud/
//#####################
//#pragma comment(linker, "/STACK:102400000,102400000")
#include <bits/stdc++.h>
using namespace std;
#define XINF INT_MAX
#define INF 0x3FFFFFFF
#define mp(X,Y) make_pair(X,Y)
#define pb(X) push_back(X)
#define rep(X,N) for(int X=0;X<N;X++)
#define rep2(X,L,R) for(int X=L;X<=R;X++)
#define dep(X,R,L) for(int X=R;X>=L;X--)
#define clr(A,X) memset(A,X,sizeof(A))
#define IT iterator
typedef long long ll;
typedef pair<int,int> PII;
typedef vector<PII> VII;
typedef vector<int> VI;
string basic_word[]={"begin","call","const","do","end","if","odd","procedure","read","then","var","while","write"};
string basic_code[]={"beginsym","callsym","constsym","dosym","endsym","ifsym","oddsym","proceduresym","readsym","thensym","varsym","whilesym","writesym"};
string cal_char[]={"+","-","*","/","=","#","<","<=",">",">=",":="};
string cal_code[]={"plus","minus","times","slash","eql","neq","lss","leq","gtr","geq","becomes"};
string sep_char[]={"(",")",",",";","."};
string sep_code[]={"lparen","rparen","comma","semicolon","period"};
map<int,string>ms;
struct Trie{
    int Next[1010][256],tail[1010];
    int root,L;
    int num = 1;
    int newnode(){
        rep(i,256)Next[L][i] = 0;
        tail[L++] = 0;
        return L - 1;
    }
    void init(){
        L = 0;
        num = 1;
        root = newnode();
    }
    int insert(string buf){
        int len = buf.length();
        int now = root;
        for(int i = 0 ; i < len ; i++){
            if(!Next[now][buf[i]])Next[now][buf[i]] = newnode();
            now = Next[now][buf[i]];
        }
        tail[now] = num;
        return num++;
    }
    void InsertIdent(string buf){
        ms[insert(buf)] = "(ident , "  + buf + ")";
        out("(ident , "  + buf + ")");
    }
    void query(string buf){
        int len = buf.length();
        int now = root;
        string identbuf = "";
        rep(i,len){
            now = Next[now][buf[i]];
            identbuf = identbuf + buf[i];
            if(now == root){
                if(isdigit(buf[i]) && identbuf.length() == 0){
                    string str = "(number,";
                    str += buf[i];
                    while(i + 1 < len){
                        if(isdigit(buf[i+1]))str = str + buf[i + 1];
                        else break;
                        i++;
                    }
                    str = str + ")";
                    out(str);
                    identbuf = "";
                }else if(isalpha(buf[i]) || isdigit(buf[i])){
                    while(i + 1 < len){
                        if(isdigit(buf[i+1])||isalpha(buf[i+1])){
                            identbuf = identbuf + buf[i + 1];
                        }else break;
                        i++;
                    }
                    InsertIdent(identbuf);
                    identbuf = "";
                }else if(buf[i] == ' ' || buf[i] == '\t')identbuf = "";
            }else if(tail[now]){
                bool flag = isdigit(buf[i]) || isalpha(buf[i]);
                if(i == len - 1 || (flag && (!isdigit(buf[i+1] && !isalpha(buf[i+1])))) || (!flag && !tail[Next[now][buf[i+1]]])){
                    string str = ms[tail[now]];
                    out(str);
                    identbuf = "";
                    now = root;
                }else if(Next[now][buf[i+1]] == root){
                    while(i + 1 < len){
                        if(isdigit(buf[i+1])||isalpha(buf[i+1])){
                            identbuf = identbuf + buf[i + 1];
                        }else break;
                        i++;
                    }
                    InsertIdent(identbuf);
                    identbuf = "";
                    now = root;
                }
            }
        }
    }
    void out(string str){
        cout << str << endl;
    }
}ac;

int main(){
    ios::sync_with_stdio(false);
    #ifdef LOCAL
        freopen("in.txt","r",stdin);
    #endif // LOCAL
    string str;
    ac.init();
    rep(i,13)ms[ac.insert(basic_word[i])] = "(" + basic_code[i] + " , "  + basic_word[i] + ")";
    rep(i,11)ms[ac.insert(cal_char[i])] = "(" + cal_code[i] + " , " + cal_char[i] + ")";
    rep(i,5)ms[ac.insert(sep_char[i])] = "(" + sep_code[i] + " , " + sep_char[i] + ")";
    while(getline(cin,str,'\n')){
        transform(str.begin(), str.end(), str.begin(), ::tolower);
        cout << "#####################" <<endl;
        cout << str <<endl;
        cout << "#####################" << endl;
        ac.query(str);
    }
    return 0;
}
```

输入文件 in.txt
```in.txt
Const num=100;
Var a1,b2;
Begin
    Read(A1);
    b2:=(+a1+1)+num;
    write(a1,b2);
end.
```
运行结果 out.txt
```
#####################
const num=100;
#####################
(constsym , const)
(ident , num)
(eql , =)
(ident , 100)
(semicolon , ;)
#####################
var a1,b2;
#####################
(varsym , var)
(ident , a1)
(comma , ,)
(ident , b2)
(semicolon , ;)
#####################
begin
#####################
(beginsym , begin)
#####################
    read(a1);
#####################
(readsym , read)
(lparen , ()
(ident , a1)
(rparen , ))
(semicolon , ;)
#####################
    b2:=(+a1+1)+num;
#####################
(ident , b2)
(becomes , :=)
(lparen , ()
(plus , +)
(ident , a1)
(plus , +)
(plus , +)
(ident , num)
(semicolon , ;)
#####################
    write(a1,b2);
#####################
(writesym , write)
(lparen , ()
(ident , a1)
(comma , ,)
(ident , b2)
(rparen , ))
(semicolon , ;)
#####################
end.
#####################
(endsym , end)
(period , .)
```
