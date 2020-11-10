---
title: R语言面向对象
date: 2020-10-20 19:51:12
tags:
	-R语言
---

#### 对象定义方式setClass()

~~~r
RGBcolor <- setClass(
  # 类名
  "RGBcolor",
  # 数据列表,用slots表示
  slots = c(r = "integer",
            g = "integer",
            b = "integer",
            label = "character"),
  #其他辅助元素，prototype参数用来设定初始值
  prototype = list(r = 0L,
                   g = 0L,
                   b = 0L,
                   label = "#00000"
  ),
  # 合法性检查
  validity = function(object){
    if(object@r<0 | object@g<0 | object@b<0){
      return("rediculous!")
    }
    if(object@r>255 | object@g>255 | object@b>255){
      return("rediculous!")
    }
    return(T)
  }
)
# 实例化一个对象
c1 <- RGBcolor(r=20L,g=20L,b=20L,label = "#8c567f")
~~~

```r
showClass("RGBcolor")
showMethods("RGBcolor")

Class "RGBcolor" [in ".GlobalEnv"]
Slots:                                           
Name:          r         g         b     label
Class:   integer   integer   integer character

Function "RGBcolor":
 <not an S4 generic function>
```

#### 通过绑定构造函数的方式进行初值的处理，绑定方法使用setMethod()函数来进行，实际上绑定的是泛型函数

```r
setMethod("initialize",signature(.Object = "RGBcolor"),
          function(.Object, r = 0L, g = 0L,b = 0L){
            .Object@r = as.integer(r)
            .Object@g = as.integer(g)
            .Object@b = as.integer(b)
			hx <- as.hexmode(c(r,g,b))
            .Object@label=paste(c("#",format(hx,width = 2)),collapse = "")
           if(.Object@r>255 | .Object@g>255 | .Object@b>255){
              cat("warning!RGB can't >255!")
             #想要停止构造对象
             # stop("rediculous") 
            }
            return(.Object)
          }                    
         )

c1 <- RGBcolor(r = 2000,g = 60 ,b = 135)
warning!RGB can't >255!
#=======================================
view(initialize)
target = new("signature", .Data = "ANY", names = ".Object", 
      package = "methods"), defined = new("signature", 
      .Data = "ANY", names = ".Object", package = "methods"), 
    generic = "initialize") 
```

#### 创建新的泛型函数(类似于抽象方法)

~~~r
setGeneric("modColor" , function(.Object,name,value) standardGeneric("modColor"))
setMethod("modColor",signature(.Object = "RGBcolor",
                               name = "character",
                               value = "numeric"),
          function(.Object,name,value){
            if(!name %in% c("r","g","b")){
              stop("rediculous!")
            }
            slot(.Object,name) <- as.integer(value)
            hx <- as.hexmode(c(.Object@r,.Object@g,.Object@b))
            .Object@label=paste(c("#",format(hx,width = 2)),collapse = "") 
           return(.Object)   
          }
) 
# 方法的重载
setMethod("modColor",signature(.Object = "RGBcolor"
                              ),
          function(.Object){
            return(.Object)
          }
          
          
          )           
#================================
c2<- RGBcolor(23,62,95)
modColor(c2,"g",34)
c2@g
c2@label 
[1] "modColor"
[1] 62
[1] "#173e5f"
c2<- modColor(c2,"g",34) 
c2@g
c2@label
[1] 34
[1] "#17225f"
~~~

#### 理解泛型函数底层原理

~~~r
hx <- as.hexmode(c(100，100，100))
paste(c("#",hx),collapse = "")
print(c("#",hx))
print(hx)
[1] "#100100100"
[1] "64" "64" "64"
[1] "#"   "100" "100" "100"
#==================================
view(print)
function (x, ...) 
UseMethod("print")
# R使用泛型函数来处理标记为不同class的数据
# 实际上我们对于某种class的数据，是调用的函数名.class名的对应函数
#==================================
class(hx)
[1] "hexmode"
#print(hx)实际上是print.hexmode(hx)
#==================================
view(print.hexmode)
function (x, ...) 
{
  print(format(x), ...)
  invisible(x)
}
#==================================
view(foramt)
function (x, ...) 
UseMethod("format")
#==================================
view(format.hexmode)
function (x, width = NULL, upper.case = FALSE, ...) 
{
  isna <- is.na(x)
  y <- as.integer(x[!isna])
  fmt0 <- if (upper.case) 
    "X"
  else "x"
  fmt <- if (!is.null(width)) 
    paste0("%0", width, fmt0)
  else paste0("%", fmt0)
  ans <- rep.int(NA_character_, length(x))
  ans0 <- sprintf(fmt, y)
  if (is.null(width) && length(y) > 1L) {
    nc <- max(nchar(ans0))
    ans0 <- sprintf(paste0("%0", nc, fmt0), y)
  }
  ans[!isna] <- ans0
  dim(ans) <- dim(x)
  dimnames(ans) <- dimnames(x)
  names(ans) <- names(x)
  ans
}
#===============================
~~~
