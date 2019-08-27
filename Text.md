Rの基本的な使い方 190917 Ryosuke TAJIMA 
==============  
# 基本
## R言語の基本的な操作
```R  
x<-2
y<-5
z<-rbind(x,y) #行で結合
v<-cbind(x,y) #列で結合
a<-c(1,3,5,7,9) #数字列
b<-c(2,4,6,8,10) #数字列
b<-c(3,6,9,12,15) #後から入れたものに更新される
c<-rbind(a,b) #行で結合
d<-cbind(a,b) #列で結合
e<-a+b #要素の足し算
f<-a*b #要素の掛け算
moji<-c("A","B","C","D") # 文字列も要素になる
length(a) #数列の長さを計算
length(moji) #数列の長さを計算
# 以下時々使う
i<- seq(1, 40, by=1) # 1つずつ
j <- seq(1,40, by=2) # 奇数のみ39まで
i2 <- rep(i,2)
i2each <- rep(i,each=2)
#こんな使い方も
i3 <- rep(i,x)
i3each <- rep(i,each=y)
```  

## ディレクトリ
### ディレクトリの確認  
```R  
getwd()  
```  
### ディレクトリの変更  
```R  
setwd()  
```  
あるいは
-  File > Change Directory ...  

## データの読み込み  
### txtファイル
```R  
d1<-read.table("Test1.txt", header=T) #txtはheaderがデフォルトでFalse
d1  
head(d1)
```  
### csvファイル
```R  
d1<-read.csv("Test1.csv") #csvはheaderがデフォルトでTrue
d1  
head(d1)
```  
  
## 読み込んだデータのチェック
### データのアウトライン
```R  
nrow(d1) # total row number  
ncol(d1) # total column number  
dim(d1) # row and column numbers  
```  
  
### 列のクラス
```R  
class(d1$Stage)  
class(d1$Rep)  
class(d1$SDW)  
# numeric -> 数値，factor -> ファクター，character -> 文字列など
```  

## データセットの一部を抽出・利用  
### 基本
```R  
A<-d1[1,1]
A<-d1[10,1]  
A<-d1[1:3,4:7]  
A<-d1[3,]  
A<-d1[,8]  
A<-d1[1:4,]  
A<-d1[,1:4]  
```  

### headerを使った抽出  
```R  
Stage<-d1$Stage  
Stage<-d1$stage # null, 大文字と小文字は区別される
```  
  
### subsetを使った抽出
```R  
sd5<-subset(d1,d1$Stage=="5week")  
sd8<-subset(d1,d1$Stage=="8week")  
```  
- データが多い場合はdplyrを使うと早い  

## データの追加
### 換算値を足す
```R  
SRratio<-d1$SDW/d1$RDW # 地上部地下部比
SRL<-d1$RLD/d1$RDW # 比根長
d2<-cbind(d1,SRratio, SRL)
```  
### ファクターを足す
- 今回は多重比較に用いるため  
  
```R  
Lavels<-c("A", "A", "A", "A", "B", "B", "B", "B", "C", "C", "C", "C", "D", "D", "D", "D", "A", "A", "A", "A", "B", "B", "B", "B", "C", "C", "C", "C", "D", "D", "D", "D")
# でも良いが
moji<-c("A","B","C","D")
Lavels<-rep(rep(moji,each=4),2)
d2<-cbind(d1,SRratio, SRL, Lavels)
```  
  
## データのアウトラインを図で確認する  
### 相関図の基本  
```R  
plot(d1[,5:8])  
plot(d1[,6], d1[,7])
plot(d1$SDW, d1$RDW)
```  
### 2つ選んで並べる  
```R  
par(mfrow=c(1,2)) # c(行, 列)  
plot(d1[,6], d1[,7])  
plot(d1$SDW, d1$RDW)  
par(mfcol=c(1,2)) # 並べ方を変えられるがここでは同じ図になる
plot(d1[,6], d1[,7])  
plot(d1$SDW, d1$RDW)  

```  

### 箱ひげ図  
```R  
boxplot(d1$SDW)  
boxplot(sd5$SDW, sd8$SDW)  
boxplot(sd5$SDW~sd5$Limitation)  
boxplot(sd5$SDW~sd5$Inoculation)  
par(mfrow=c(1,2))
boxplot(sd5$SDW~sd5$Limitation)  
boxplot(sd8$SDW~sd8$Limitation)  
```  
  
# 統計解析  
## 分散分析  
### 一元配置の分散分析  
```R  
result<-aov(SDW~Limitation, sd5)  
summary(result)  
```  
  
### 二元配置の分散分析  
```R  
result<-aov(SDW~Limitation+Inoculation+Limitation*Inoculation, sd5)  
summary(result)  
result<-aov(SDW~Limitation*Inoculation, sd5) # 省略記法  
summary(result)  
```  
  
### 乱塊法
```R  
result<-aov(SDW~Rep+Limitation*Inoculation, sd5)
summary(result)  
```  
  
### 分割区法
```R  
result<-aov(SDW~Rep+Limitation+Error(Rep/Limitation)+Inoculation+Limitation*Inoculation, sd5)   
summary(result)  
```  
  
### 二方分割法
```R  
result<-aov(SDW~Rep+Limitation+Inoculation+Error(Rep/(Limitation*Inoculation))+Limitation*Inoculation, sd5)   
summary(result)  
```  
  
## 多重比較
### TukeyHSD  
```R  
sd25<-subset(d2,d2$Stage=="5week")  #データの切り分け
sd28<-subset(d2,d2$Stage=="8week")  
result<-aov(SDW~Lavels, sd25) # 一元配置の分散分析
summary(result)  
TukeyHSD(result)  
```  
  
### multcomp利用，Tukey  
```R  
library(multcomp)
result<-aov(SDW~Lavels, sd25) # 一元配置の分散分析
Tukey<-glht(result, linfct=mcp(Lavels="Tukey"))  
summary(Tukey)  
cld(Tukey, level = 0.05, decreasing = TRUE) # アルファベットをつける
```  

### multcomp利用，Dunnett  
```R  
library(multcomp)
result<-aov(SDW~Lavels, sd25) # 一元配置の分散分析
Dunnett<-glht(result, linfct=mcp(Lavels="Dunnett"))  
summary(Dunnett)  
```  
  
## 相関  
```R  
cor(sd5$SDW, sd5$SPC)  
allcor<-cor(sd5[,5:8]) #まとめて解析  
allcor
```  
  
## 回帰分析
```R  
result<-lm(SDW~SPC, sd5)  
summary(result)  
```  
  
## 重回帰分析
```R  
allcor # 事前に多重共線性のチェック
result<-lm(SDW~SPC+RLD, sd5)  
summary(result)  
```  

# 図表作成
## 基本  
```R  
a<-c(1,3,5,7,9) #数字列
b<-c(2,4,6,8,10) #数字列
barplot(a) # 棒グラフ
plot(a,b) #散布図
plot(a,b, type="l") #線にする b：点と線，o：点と線重ねる
plot(a,b, col="red") #基本的な色はだいたいある．
plot(a,b, pch=0) #シンボルの形を変える．16をよく使う
plot(a,b, cex=2) #サイズの変更
plot(a,b, type="o", col="red", pch=16, cex=2) #組み合わせられる
```  

## 実際のデータで作成
```R  
plot(sd25$SDW,sd25$RDW, 
        xlim = c(0,1),  # x軸の境界
        ylim = c(0,0.5), # y軸の境界
        xlab = "ShootDW", # x軸ラベル
        ylab = "RootDW", # y軸ラベル
        col = "red",
        pch=16,
        cex=1.5,
    )
```  
## それなりの図
- こうなってくると手書きに近い  
  
```R  
plot(sd25$SDW,sd25$RDW, 
        xlim = c(0,1),  # x軸の境界
        ylim = c(0,0.5), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "red",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.2,0.4,0.6,0.8,1.0), cex.axis=1.2, las=1, labels = T) # x軸目盛
axis(2, at=c(0,0.1,0.2,0.3,0.4,0.5), cex.axis=1.2, las=1, labels = T) # y軸目盛
mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5) # x軸ラベル
mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 2.5) #y軸ラベル
box("plot",lty=1,lwd=1) # 枠をつける
```  
  
## 重ねる
```R  
plot(sd25$SDW,sd25$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "red",
        pch=16,
        cex=2,
        axes=FALSE
    )
par(new=TRUE) # デフォルトはFalseなので図は更新される
plot(sd28$SDW,sd28$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "blue",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.6,1.2,1.8,2.4), cex.axis=1.2, las=1, labels = T) # X軸ラベル
axis(2, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # y軸ラベル
mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 2.5)
box("plot",lty=1,lwd=1) # 枠をつける
```  

## 並べる

```R  
par(mfrow=c(1,2)) # c(行, 列)  
plot(sd25$SDW,sd25$RDW, 
        xlim = c(0,1.2),  # x軸の境界
        ylim = c(0,0.6), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "red",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # X軸ラベル
axis(2, at=c(0,0.2,0.4,0.6), cex.axis=1.2, las=1, labels = T) # y軸ラベル
mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 2.5)
box("plot",lty=1,lwd=1) # 枠をつける


plot(sd28$SDW,sd28$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "blue",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.6,1.2,1.8,2.4), cex.axis=1.2, las=1, labels = T) # X軸ラベル
axis(2, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # y軸ラベル
mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 2.5)
box("plot",lty=1,lwd=1) # 枠をつける
```  

## 少し変わった並べ方  
```R  
par(oma = c(2, 2, 2, 2)) # 下・左・上・右

par(mfrow=c(1,2)) # c(行, 列)  
par(mar = c(5, 5, 0, 0))  # 下・左・上・右
plot(sd25$SDW,sd25$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "red",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.6,1.2,1.8,2.4), cex.axis=1.2, las=1, labels = T) # X軸ラベル
axis(2, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # y軸ラベル
# mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 4)
box("plot",lty=1,lwd=1) # 枠をつける

par(mar = c(5, 0, 0, 5))  # 下・左・上・右
plot(sd28$SDW,sd28$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "blue",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.6,1.2,1.8,2.4), cex.axis=1.2, las=1, labels = T) # X軸ラベル
# axis(2, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # y軸ラベル
# mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
# mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 2.5)
box("plot",lty=1,lwd=1) # 枠をつける
mtext("ShootDW",adj=0.5,side = 1, cex = 1.5, line = -1, outer=TRUE)

```  

## pdfで書き出す
###  基本

```R  
pdf("test.pdf", width=5, height=5)
plot(a,b, type="o", col="red", pch=16, cex=2) #組み合わせられる
dev.off()
```  

###  実際
```R  
pdf("Fig.pdf", width=10, height=5)
par(oma = c(2, 2, 2, 2)) # 下・左・上・右

par(mfrow=c(1,2)) # c(行, 列)  
par(mar = c(5, 5, 0, 0))  # 下・左・上・右
plot(sd25$SDW,sd25$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "red",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.6,1.2,1.8,2.4), cex.axis=1.2, las=1, labels = T) # X軸ラベル
axis(2, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # y軸ラベル
# mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 4)
box("plot",lty=1,lwd=1) # 枠をつける

par(mar = c(5, 0, 0, 5))  # 下・左・上・右
plot(sd28$SDW,sd28$RDW, 
        xlim = c(0,2.4),  # x軸の境界
        ylim = c(0,1.2), # y軸の境界
        xlab = "", # x軸ラベル
        ylab = "", # y軸ラベル
        col = "blue",
        pch=16,
        cex=2,
        axes=FALSE
    )
axis(1, at=c(0,0.6,1.2,1.8,2.4), cex.axis=1.2, las=1, labels = T) # X軸ラベル
# axis(2, at=c(0,0.3,0.6,0.9,1.2), cex.axis=1.2, las=1, labels = T) # y軸ラベル
# mtext(1, text="ShootDW", adj=0.5, cex = 1.5, line = 2.5)
# mtext(2, text="RootDW", adj=0.5, cex = 1.5, line = 2.5)
box("plot",lty=1,lwd=1) # 枠をつける
mtext("ShootDW",adj=0.5,side = 1, cex = 1.5, line = -1, outer=TRUE)
dev.off()
```  

# さらに便利に
## コードはソースとして読むこともできる
```R  
source("Fig1.r")
```  

## 繰り返しや分岐の関数を利用する
### for  
```R  
d<-0
for (i in 1:100) {
    d<-d+i
}

```  

### forとifの組み合わせ  
```R  
do<-0
de<-0
for (i in 1:100) {
    if (i%%2==0) {
        de<-de+i
    } else {
        do<-do+i
    }
}
```  

# 図表作成発展的課題
  
- [Fig2](https://github.com/blukaniro/TrainingR190917/blob/master/Fig2.pdf)を作る
- [ソースコード](https://github.com/blukaniro/TrainingR190917/blob/master/Fig2.r)
- [for, ifを使わないソースコード](https://github.com/blukaniro/TrainingR190917/blob/master/Fig2long.r)



# おしまい．お疲れさまでした．