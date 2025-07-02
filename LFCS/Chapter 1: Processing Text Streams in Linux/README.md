# Chapter 1: Processing Text Streams in Linux
Having previous experience with Linux I understand how different operators work <br>
The only reason I didn't skip this part was because I was curious about some parts <br>

## pipe operator
Takes output from one program to input to other program without creating file descriptor. <br>
```
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
[mateusz@MateuszLaptop ~]$ 
```

## ">" operator
Writes to file or block device, if file exists it overwrites the content. <br>
```
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum > testfile.txt
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum > testfile.txt
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum > testfile.txt
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
[mateusz@MateuszLaptop ~]$ 
```

Wipe disk <br>

```cat /dev/null > /dev/sdX```

## ">>" operator
Writes to file or block device, if file exists appends the content. 
```
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum >> testfile.txt
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum >> testfile.txt
[mateusz@MateuszLaptop ~]$ echo "wat" | sha512sum >> testfile.txt
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
[mateusz@MateuszLaptop ~]$ 
```

## "<" operator
Reads file or block device and provides STDIN. 
```
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
wat
[mateusz@MateuszLaptop ~]$ sha512sum < testfile.txt 
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
[mateusz@MateuszLaptop ~]$ 
```
## Streams
STDIN input to program, has number of 0 <br>
STDOUT output of a program, has number of 1 <br>
STDERR error of a program, has a number of 2 <br>
It's possible to chain and redirect each of them to different files. <br>
```
[mateusz@MateuszLaptop ~]$ sha512sum < testfile.txt 1>>out.txt 2>>error.txt
[mateusz@MateuszLaptop ~]$ sha512sum -asdjasdlaskjdaslkj < testfile.txt 1>>out.txt 2>>error.txt
[mateusz@MateuszLaptop ~]$ cat out.txt 
dc5534422ab49283655b52f6dd7f0f3caad475fc1eaaccc84ceadb20dee2553c55cd079abefaa43fd6b268e11324fad79794ae680c351c9b76e160e1a949f968  -
[mateusz@MateuszLaptop ~]$ cat error.txt 
sha512sum: invalid option -- 'a'
Try 'sha512sum --help' for more information.
[mateusz@MateuszLaptop ~]$ 
```
## sed command
sed can be used to replace text in a file. <br>
```
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
replaceThat1

ASDASD

WWWWWWWWW
[mateusz@MateuszLaptop ~]$ sed -i 's/replaceThat1/ireplacedit/g' testfile.txt 
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
ireplacedit

ASDASD

WWWWWWWWW
[mateusz@MateuszLaptop ~]$ 
```

You can remove empty lines. <br>
```
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
ireplacedit

ASDASD

WWWWWWWWW
[mateusz@MateuszLaptop ~]$ sed -ri '/^\s*$/d' testfile.txt 
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
ireplacedit
ASDASD
WWWWWWWWW
[mateusz@MateuszLaptop ~]$ 
```

Additionally, it supports regex. <br>

```
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
ireplacedit
ASDASD
WWWWWWWWW
[mateusz@MateuszLaptop ~]$ sed -i 's/[W]\{5\}/replaced5only/g' testfile.txt 
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
ireplacedit
ASDASD
replaced5onlyWWWW
[mateusz@MateuszLaptop ~]$ 
```

## uniq command
Detects duplicate lines, but only if they are adjacent <br>
```
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
1
1
1
2
2
2
3
3
3
3
4
4
4
1
1
1
1
[mateusz@MateuszLaptop ~]$ uniq testfile.txt 
1
2
3
4
1
[mateusz@MateuszLaptop ~]$ 
```

## grep command
Allows to find text in a file. Also works with regex. <br>
```
[mateusz@MateuszLaptop ~]$ cat testfile.txt 
1
1
1
2
2
2
3
3
3
3
4
4
4
1
1
1
1
[mateusz@MateuszLaptop ~]$ cat testfile.txt | grep 1 
1
1
1
1
1
1
1
[mateusz@MateuszLaptop ~]$ 
```

## Count amount of lines
This is just an example, but I like to use this command to count lines of code in my projects <br>
```
[mateusz@MateuszLaptop GNS3_Networks]$ find . -name "*.md" -exec cat {} + | wc -l
1154
[mateusz@MateuszLaptop GNS3_Networks]$ 
```