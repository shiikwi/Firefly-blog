---
title: Siglus引擎AlphaROM动态bypass记录
published: 2026-07-29
description: 某古董的siglus AlphaROM，区域验证，dvd校验bypass记录
tags: [gal逆向, 动态调试]
category: gal逆向
slug: siglus-bypass
draft: false
---

最近整理旧物的时候遇到了AlphaROM，老古董drm了，网上搜一搜资料也很多。dira有写过dll劫持的工具，azure大佬写过通用crack。除了早些时候siglus，reallive和戏画用这个，好像也没啥人用了，现在应该是被淘汰了。  
既然遇到了，那就顺便看一看好了，单纯用动态调试过掉校验运行游戏。（其实就是懒得写hook代码）校验弹窗就长下面这样子  
![](https://img.koisukui.top/file/alpharombypass/AlphaROMSample.png)

# 过AlphaROM
上x64dbg，开不开反反调试其实没差，反正这反调试四处漏风。先随便打几个断点，然后跑起来看看情况，发现它有时候会报一个错误，说什么sarcheck.dll加载失败，那不就相当于自爆了吗。再一看区段.detour。行了，那就常规思路，先断‌VirtualAlloc，LoadLibrary，GetProcAddress等几个常见的看看。很快就发现‌VirtualAlloc能捕捉到sarcheck.dll的加载。  
![](https://img.koisukui.top/file/alpharombypass/virtualalloc.png)
返回到exe领空后来到下面这个函数，简单说就是个PE Loader。用Process Monitor观察下，这个程序没有读取或释放外部的sarcheck.dll。那么就一定是内联在exe中了。再往上追一层看下。
```asm
push ebp
mov ebp,esp
sub esp,10
pushad 
mov dword ptr ss:[ebp-10],0
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+4]
test eax,eax
je siglusengine.4CF7980
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+14]
test eax,eax
jne siglusengine.4CF7980
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+4]
cmp word ptr ds:[eax],5A4D
jne siglusengine.4CF7980
mov ebx,dword ptr ds:[eax+3C]
add eax,ebx
cmp dword ptr ds:[eax],4550
jne siglusengine.4CF7980
mov ebx,dword ptr ds:[eax+34]
mov dword ptr ss:[ebp-C],ebx
mov ebx,dword ptr ds:[eax+50]
mov dword ptr ss:[ebp-8],ebx
push edi
push 4
push 2000
push dword ptr ss:[ebp-8]
push dword ptr ss:[ebp-C]
call dword ptr ds:[edi+D4732DA]
pop edi
test eax,eax
jne siglusengine.4CF7973
push edi
push 4
push 2000
push dword ptr ss:[ebp-8]
push 0
call dword ptr ds:[edi+D4732DA]
pop edi
test eax,eax
je siglusengine.4CF7980
mov ebx,dword ptr ss:[ebp+8]
mov dword ptr ds:[ebx+14],eax
mov dword ptr ss:[ebp-10],1
popad 
mov eax,dword ptr ss:[ebp-10]
leave 
ret 4
```

同样是PE Loader，作为上级分发函数，它会在完成dll的解析后获得绝对地址，并在末尾调用它来初始化。在倒数第二个call，`call siglusengine.4CF7E76`打个断点，跟入看下。
```asm
push ebp
mov ebp,esp
sub esp,8
pushad 
mov dword ptr ss:[ebp-8],0
mov eax,dword ptr ss:[ebp+8]
mov ebx,dword ptr ds:[eax+10]
test ebx,ebx
je siglusengine.4CF80B5
lea eax,dword ptr ds:[eax+10]
inc dword ptr ds:[eax]
jmp siglusengine.4CF81CB
cmp dword ptr ds:[edi+D470965],32303031
jne siglusengine.4CF80CD
push dword ptr ss:[ebp+8]
call siglusengine.4CF805B
push dword ptr ss:[ebp+8]
call siglusengine.4CF78E2
test eax,eax
je siglusengine.4CF81CB
mov eax,dword ptr ss:[ebp+8]
mov dword ptr ds:[eax+10],1
mov ebx,dword ptr ss:[ebp+C]
mov dword ptr ds:[eax+18],ebx
push dword ptr ss:[ebp+8]
call siglusengine.4CF7988
test eax,eax
je siglusengine.4CF81B9
push dword ptr ss:[ebp+8]
call siglusengine.4CF7A05
test eax,eax
je siglusengine.4CF81B9
push dword ptr ss:[ebp+8]
call siglusengine.4CF7BC1
mov edx,0
test edx,edx
je siglusengine.4CF812D
push dword ptr ss:[ebp+8]
call siglusengine.4CF7AC0
mov dword ptr ss:[ebp-8],eax
push dword ptr ss:[ebp+8]
call siglusengine.4CF7D5B
test eax,eax
je siglusengine.4CF81B9
call siglusengine.4CFD9F9
cmp dword ptr ds:[edi+D470965],32303031
jne siglusengine.4CF815A
push dword ptr ss:[ebp+8]
call siglusengine.4CF805B
push dword ptr ss:[ebp+8]
call siglusengine.4CF7CD2
test eax,eax
je siglusengine.4CF81B9
cmp dword ptr ds:[edi+D470965],32303031
jne siglusengine.4CF818A
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+14]
mov edx,edi
push 0
push eax
call siglusengine.4D00C61
mov edx,0
test edx,edx
je siglusengine.4CF81A7
mov edx,edi
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+14]
push 0
push eax
call siglusengine.4D0098A
push 1
push dword ptr ss:[ebp+8]
call siglusengine.4CF7E76
test eax,eax
jne siglusengine.4CF81CB
mov eax,dword ptr ss:[ebp+8]
mov dword ptr ds:[eax+18],1
push dword ptr ds:[eax+14]
call siglusengine.4CF87B3
popad 
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+14]
leave 
ret 8
```

这里就是dll的调用入口了，简单检查MZ标志后，加载基址，通过`call eax`调用
```asm
push ebp
mov ebp,esp
sub esp,4
pushad 
mov dword ptr ss:[ebp-4],1
mov eax,dword ptr ss:[ebp+8]
mov eax,dword ptr ds:[eax+18]
test eax,3
jne siglusengine.4CF7EF1
mov eax,dword ptr ss:[ebp+8]
mov ecx,dword ptr ds:[eax+8]
mov eax,dword ptr ds:[eax+4]
cmp dword ptr ds:[edi+D470965],32303031
jne siglusengine.4CF7EBE
cmp word ptr ds:[eax],5A4D
je siglusengine.4CF7EBE
call siglusengine.4CFDB1E
cmp word ptr ds:[eax],5A4D
je siglusengine.4CF7ECE
call siglusengine.4CFDB1E
mov ebx,dword ptr ds:[eax+3C]
add eax,ebx
mov eax,dword ptr ds:[eax+28]
test eax,eax
je siglusengine.4CF7EF1
mov ebx,dword ptr ss:[ebp+8]
mov ebx,dword ptr ds:[ebx+14]
add eax,ebx
push 0
push dword ptr ss:[ebp+C]
push ebx
call eax
mov dword ptr ss:[ebp-4],eax
popad 
mov eax,dword ptr ss:[ebp-4]
leave 
ret 8
```

然后就是这个dll的开头了，这里就放一部分了。它会初始化两个key，然后去xor循环解密。要研究校验逻辑的话可以再跟下，我是懒得看了。其实它这个逻辑和那个dmm game player的drm差不多，就是Ai5Win那个引擎常用的那玩意。思路很简单，就是直接让它返回，不要运行这个dll就可以了。
```asm
sub esp,4
push eax
push ebx
call 1057400B
int3 
pop eax
mov ebx,eax
inc eax
sub eax,283000
sub eax,1009CF7F
add eax,1009CF74
cmp byte ptr ds:[ebx],CC
jne 1057403C
mov byte ptr ds:[ebx],0
mov ebx,1000
push 7AC71553
push 19513D01
push ebx
push eax
call 10574046
add eax,0
mov dword ptr ss:[esp+8],eax
pop ebx
pop eax
ret 
```

由于函数调用约定，DllMain会固定接收三个参数。所以这个dll被调用的时候，已经压入了12字节的参数，现在要让它返回，就必须用`ret 0xC`来保持堆栈平衡。把函数开头的`sub esp,4`改成这个，然后跑起来，启动！
![](https://img.koisukui.top/file/alpharombypass/1785334089154_image.png)
好吧，还有第二关。


# 过区域验证
siglus的区域验证，最常见的就是去断MessageBox，然后找`call cmp je`这样的结构，把它们nop掉。不过套了AlphaROM，带壳之后msgbox的效果就不太好了，软断可能会触发反调，并且还会将一部分api函数复制到自己的内存中，执行时调用复制的代码执行。就算执行原始api，跳板跳回去的时候也大概率不会在函数头上，所以硬断的效果也不太好。  
这样的话就换种方法，直接搜字符串好了。在过掉sarcheck的校验后，还是断住‌VirtualAlloc，f9几下，让它把代码释放过去，然后搜索主程序的字符串`TOKYO`跳过去后对着函数头ctrl+r一下，只有一个调用。
![](https://img.koisukui.top/file/alpharombypass/1785335120725_image.png)
这里就很熟悉了，经典三重校验，系统语言，语言区域，时区。继续ctrl+r找一下这个函数的上级调用。
![](https://img.koisukui.top/file/alpharombypass/1785335607638_image.png)
直接把`call test je`三条一起nop掉，然后启动！
![](https://img.koisukui.top/file/alpharombypass/1785335973928_image.png)
经典，还有第三关


# 过dvd校验
这个游戏的dvd校验稍微有点特殊，它有点像是马戏团社，就是dc领域大神用的那个dvd校验逻辑。这个最好的方法是断GetDriveType，我试过msg效果并不好，离校验位置有点远。字符串的话给内存下硬断跟几层虽然也能找到，但是都不如GetDriveType来的直接。
```asm
push ebp
mov ebp,esp
push FFFFFFFF
push siglusengine.6EC643
mov eax,dword ptr fs:[0]
push eax
sub esp,9C
mov eax,dword ptr ds:[795490]
xor eax,ebp
mov dword ptr ss:[ebp-10],eax
push ebx
push esi
push edi
push eax
lea eax,dword ptr ss:[ebp-C]
mov dword ptr fs:[0],eax
xor edi,edi
push 1
mov dword ptr ss:[ebp-A4],edi
call dword ptr ds:[713208]
mov dword ptr ss:[ebp-A8],eax
mov dword ptr ss:[ebp-A0],41
lea ebx,dword ptr ds:[edi+8]
mov esi,7
xor eax,eax
mov dword ptr ss:[ebp-50],esi
mov dword ptr ss:[ebp-54],edi
mov word ptr ss:[ebp-64],ax
mov eax,dword ptr ss:[ebp-A0]
lea ecx,dword ptr ss:[ebp-64]
push ecx
lea ecx,dword ptr ss:[ebp-80]
mov dword ptr ss:[ebp-4],edi
call siglusengine.4631C0
push eax
mov edx,siglusengine.75A140
lea eax,dword ptr ss:[ebp-9C]
mov byte ptr ss:[ebp-4],1
call siglusengine.463200
add esp,8
push edi
mov byte ptr ss:[ebp-4],2
xor edx,edx
push eax
or eax,FFFFFFFF
lea ecx,dword ptr ss:[ebp-2C]
mov dword ptr ss:[ebp-18],esi
mov dword ptr ss:[ebp-1C],edi
mov word ptr ss:[ebp-2C],dx
call siglusengine.402000
cmp dword ptr ss:[ebp-88],ebx
jb siglusengine.499323
mov eax,dword ptr ss:[ebp-9C]
push eax
call siglusengine.5CA445
add esp,4
xor ecx,ecx
mov dword ptr ss:[ebp-88],esi
mov dword ptr ss:[ebp-8C],edi
mov word ptr ss:[ebp-9C],cx
cmp dword ptr ss:[ebp-6C],ebx
jb siglusengine.499349
mov edx,dword ptr ss:[ebp-80]
push edx
call siglusengine.5CA445
add esp,4
xor eax,eax
mov dword ptr ss:[ebp-6C],esi
mov dword ptr ss:[ebp-70],edi
mov word ptr ss:[ebp-80],ax
mov byte ptr ss:[ebp-4],6
cmp dword ptr ss:[ebp-50],ebx
jb siglusengine.49936A
mov ecx,dword ptr ss:[ebp-64]
push ecx
call siglusengine.5CA445
add esp,4
mov eax,dword ptr ss:[ebp-2C]
xor edx,edx
mov dword ptr ss:[ebp-50],esi
mov dword ptr ss:[ebp-54],edi
mov word ptr ss:[ebp-64],dx
cmp dword ptr ss:[ebp-18],ebx
jae siglusengine.499381
lea eax,dword ptr ss:[ebp-2C]
push eax
call dword ptr ds:[<&JMP.&GetDriveTypeW>]
cmp eax,5
jne siglusengine.4993D0
mov ecx,dword ptr ds:[7B2470]
lea eax,dword ptr ss:[ebp-2C]
add ecx,20
push eax
lea esi,dword ptr ss:[ebp-48]
call siglusengine.409520
mov eax,dword ptr ss:[ebp-48]
add esp,4
cmp dword ptr ss:[ebp-34],ebx
jae siglusengine.4993AF
mov eax,esi
push eax
call dword ptr ds:[<&JMP.&GetFileAttributesW>]
cmp eax,FFFFFFFF
je siglusengine.4993BF
test al,10
je siglusengine.499401
cmp dword ptr ss:[ebp-34],ebx
jb siglusengine.4993D0
mov ecx,dword ptr ss:[ebp-48]
push ecx
call siglusengine.5CA445
add esp,4
mov dword ptr ss:[ebp-4],FFFFFFFF
cmp dword ptr ss:[ebp-18],ebx
jb siglusengine.4993E8
mov edx,dword ptr ss:[ebp-2C]
push edx
call siglusengine.5CA445
add esp,4
mov eax,dword ptr ss:[ebp-A0]
inc eax
mov dword ptr ss:[ebp-A0],eax
cmp ax,5A
jbe siglusengine.4992B1
jmp siglusengine.499440
movzx eax,word ptr ss:[ebp-A0]
mov dword ptr ss:[ebp-A4],eax
cmp dword ptr ss:[ebp-34],ebx
jb siglusengine.49941F
mov ecx,dword ptr ss:[ebp-48]
push ecx
call siglusengine.5CA445
add esp,4
xor edx,edx
mov dword ptr ss:[ebp-34],7
mov dword ptr ss:[ebp-38],edi
mov word ptr ss:[ebp-48],dx
cmp dword ptr ss:[ebp-18],ebx
jb siglusengine.499440
mov eax,dword ptr ss:[ebp-2C]
push eax
call siglusengine.5CA445
add esp,4
mov ecx,dword ptr ss:[ebp-A8]
push ecx
call dword ptr ds:[713208]
mov ax,word ptr ss:[ebp-A4]
mov ecx,dword ptr ss:[ebp-C]
mov dword ptr fs:[0],ecx
pop ecx
pop edi
pop esi
pop ebx
mov ecx,dword ptr ss:[ebp-10]
xor ecx,ebp
call siglusengine.5CA436
mov esp,ebp
pop ebp
ret 
```
它会开始遍历A-Z盘，检查不是不是光驱，如果是的话，再在盘下校验一个文件，然后把盘符返回。对这个游戏来说，在这个函数patch不太好，就算把校验跳过了，它最后也还会把错误的盘符返回回去，上级函数会拿这个盘符和`":\\dummy" `进行拼接，然后对这个文件进行读取校验。  
对它ctrl+r一下，可能会搜到两个调用，找有`":\\dummy" `这个字符串的那个函数，另一个是注册表里的校验。
```asm
push ebp
mov ebp,esp
push FFFFFFFF
push siglusengine.6F8E4C
mov eax,dword ptr fs:[0]
push eax
sub esp,F0
mov eax,dword ptr ds:[795490]
xor eax,ebp
mov dword ptr ss:[ebp-10],eax
push ebx
push esi
push edi
push eax
lea eax,dword ptr ss:[ebp-C]
mov dword ptr fs:[0],eax
mov eax,dword ptr ss:[ebp+10]
mov dword ptr ss:[ebp-F8],eax
call siglusengine.499260
movzx edi,ax
xor esi,esi
cmp di,si
je siglusengine.462D1A
mov ebx,7
xor ecx,ecx
mov dword ptr ss:[ebp-50],ebx
mov dword ptr ss:[ebp-54],esi
mov word ptr ss:[ebp-64],cx
lea edx,dword ptr ss:[ebp-64]
push edx
mov eax,edi
lea ecx,dword ptr ss:[ebp-80]
mov dword ptr ss:[ebp-4],esi
call siglusengine.4631C0
add esp,4
push eax
mov edx,siglusengine.756D60
lea eax,dword ptr ss:[ebp-9C]
mov byte ptr ss:[ebp-4],1
call siglusengine.463200
add esp,4
mov byte ptr ss:[ebp-4],2
xor ecx,ecx
push esi
mov word ptr ss:[ebp-48],cx
push eax
or eax,FFFFFFFF
lea ecx,dword ptr ss:[ebp-48]
mov dword ptr ss:[ebp-34],ebx
mov dword ptr ss:[ebp-38],esi
call siglusengine.402000
cmp dword ptr ss:[ebp-88],8
jb siglusengine.462B0C
mov edx,dword ptr ss:[ebp-9C]
push edx
call siglusengine.5CA445
add esp,4
xor eax,eax
mov dword ptr ss:[ebp-88],ebx
mov dword ptr ss:[ebp-8C],esi
mov word ptr ss:[ebp-9C],ax
cmp dword ptr ss:[ebp-6C],8
jb siglusengine.462B33
mov ecx,dword ptr ss:[ebp-80]
push ecx
call siglusengine.5CA445
add esp,4
xor edx,edx
mov dword ptr ss:[ebp-6C],ebx
mov dword ptr ss:[ebp-70],esi
mov word ptr ss:[ebp-80],dx
mov byte ptr ss:[ebp-4],6
cmp dword ptr ss:[ebp-50],8
jb siglusengine.462B55
mov eax,dword ptr ss:[ebp-64]
push eax
call siglusengine.5CA445
add esp,4
mov eax,dword ptr ss:[ebp-48]
xor ecx,ecx
cmp dword ptr ss:[ebp-34],8
mov dword ptr ss:[ebp-50],ebx
mov dword ptr ss:[ebp-54],esi
mov word ptr ss:[ebp-64],cx
jae siglusengine.462B6D
lea eax,dword ptr ss:[ebp-48]
push eax
call dword ptr ds:[<&JMP.&GetFileAttributesW>]
cmp eax,FFFFFFFF
je siglusengine.462B91
test al,10
je siglusengine.462B91
mov dword ptr ss:[ebp-F4],2
mov edi,8
jmp siglusengine.462CA8
xor edx,edx
mov dword ptr ss:[ebp-F4],1
mov dword ptr ss:[ebp-A4],ebx
mov dword ptr ss:[ebp-A8],esi
mov word ptr ss:[ebp-B8],dx
lea eax,dword ptr ss:[ebp-B8]
push eax
mov eax,edi
lea ecx,dword ptr ss:[ebp-D4]
mov byte ptr ss:[ebp-4],bl
call siglusengine.4631C0
add esp,4
push eax
mov edx,siglusengine.756D70
lea eax,dword ptr ss:[ebp-F0]
mov byte ptr ss:[ebp-4],8
call siglusengine.463200
add esp,4
mov byte ptr ss:[ebp-4],9
xor ecx,ecx
push esi
mov word ptr ss:[ebp-2C],cx
push eax
or eax,FFFFFFFF
lea ecx,dword ptr ss:[ebp-2C]
mov dword ptr ss:[ebp-18],ebx
mov dword ptr ss:[ebp-1C],esi
call siglusengine.402000
push esi
lea edx,dword ptr ss:[ebp-2C]
push edx
or eax,FFFFFFFF
lea ecx,dword ptr ss:[ebp-48]
mov byte ptr ss:[ebp-4],A
call siglusengine.402000
mov edi,8
cmp dword ptr ss:[ebp-18],edi
jb siglusengine.462C29
mov eax,dword ptr ss:[ebp-2C]
push eax
call siglusengine.5CA445
add esp,4
xor ecx,ecx
mov dword ptr ss:[ebp-18],ebx
mov dword ptr ss:[ebp-1C],esi
mov word ptr ss:[ebp-2C],cx
cmp dword ptr ss:[ebp-DC],edi
jb siglusengine.462C4C
mov edx,dword ptr ss:[ebp-F0]
push edx
call siglusengine.5CA445
add esp,4
xor eax,eax
mov dword ptr ss:[ebp-DC],ebx
mov dword ptr ss:[ebp-E0],esi
mov word ptr ss:[ebp-F0],ax
cmp dword ptr ss:[ebp-C0],edi
jb siglusengine.462C78
mov ecx,dword ptr ss:[ebp-D4]
push ecx
call siglusengine.5CA445
add esp,4
xor edx,edx
mov byte ptr ss:[ebp-4],6
mov dword ptr ss:[ebp-C0],ebx
mov dword ptr ss:[ebp-C4],esi
mov word ptr ss:[ebp-D4],dx
cmp dword ptr ss:[ebp-A4],edi
jb siglusengine.462CA8
mov eax,dword ptr ss:[ebp-B8]
push eax
call siglusengine.5CA445
add esp,4
mov ecx,dword ptr ss:[ebp+C]
mov eax,dword ptr ss:[ebp-F4]
push ecx
mov ecx,dword ptr ss:[ebp+8]
lea edx,dword ptr ss:[ebp-48]
push edx
mov edx,dword ptr ss:[ebp-F8]
push eax
call siglusengine.548CD0
mov dword ptr ss:[ebp-1C],esi
mov dword ptr ss:[ebp-18],ebx
lea esi,dword ptr ss:[ebp-2C]
test al,al
jne siglusengine.462D1E
xor ecx,ecx
push 13
mov eax,siglusengine.756D74
mov word ptr ss:[ebp-2C],cx
call siglusengine.4023E0
mov byte ptr ss:[ebp-4],B
mov edx,dword ptr ds:[7B24C4]
push 5
push edx
mov ecx,esi
call siglusengine.42BC80
cmp dword ptr ss:[ebp-18],edi
jb siglusengine.462D09
mov eax,dword ptr ss:[ebp-2C]
push eax
call siglusengine.5CA445
add esp,4
cmp dword ptr ss:[ebp-34],edi
jb siglusengine.462D1A
mov ecx,dword ptr ss:[ebp-48]
push ecx
call siglusengine.5CA445
add esp,4
xor al,al
jmp siglusengine.462D68
xor edx,edx
push 12
mov eax,siglusengine.756D9C
mov word ptr ss:[ebp-2C],dx
call siglusengine.4023E0
mov byte ptr ss:[ebp-4],C
mov eax,dword ptr ds:[7B24C4]
push 5
push eax
lea ecx,dword ptr ss:[ebp-2C]
call siglusengine.42BC80
cmp dword ptr ss:[ebp-18],edi
jb siglusengine.462D55
mov ecx,dword ptr ss:[ebp-2C]
push ecx
call siglusengine.5CA445
add esp,4
cmp dword ptr ss:[ebp-34],edi
jb siglusengine.462D66
mov edx,dword ptr ss:[ebp-48]
push edx
call siglusengine.5CA445
add esp,4
mov al,1
mov ecx,dword ptr ss:[ebp-C]
mov dword ptr fs:[0],ecx
pop ecx
pop edi
pop esi
pop ebx
mov ecx,dword ptr ss:[ebp-10]
xor ecx,ebp
call siglusengine.5CA436
mov esp,ebp
pop ebp
ret 
```
这个就是校验函数了，直接看最后的成功分支就行了，`test al, al` `al=0`是dummy校验失败， `al=1`成功。所以直接在函数开头打桩就行了。
```asm
mov al,1
ret
```
![](https://img.koisukui.top/file/alpharombypass/1785338058155_image.png)
这样改的好处是，他还得谢谢咱呢。


# 结语
脱壳有时间再说吧，反正这东西现在也没啥人用了。有的是套的tmd，那个可以直接用[Magicmida
](https://github.com/Hendi48/Magicmida)脱，不过我这个样本不是。总之先这样了，懒得写hook代码了。
