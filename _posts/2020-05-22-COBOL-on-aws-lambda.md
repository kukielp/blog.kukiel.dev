---
layout: post
title:  "Runnign Cobol on AWS Lambda"
date:   2020-05-22 018:18:12 +1000
categories: aws COBOL lambda ec2 costsavings docker
---

<img src="https://cdn.kukiel.dev/cobol-book.jpg" alt="Cobol" style="width:200px;
    display: block;
    margin-left: auto;
    margin-right: auto;"/>


I have spent part of my working career in Financial services, it's a great industry and an anyone who has spent any time there knows Mainframe, COBOL and Fortran do come up.  I know of one COBOL programmer personally ( the wife of a friend ), how ever have never see COBOL code myself unitl ~a year ago where I did some research, found a compiler and tried it (Hello CBOLO).  I then did a quick google about running this in Lambda ( why not? ) and found this [repo](https://github.com/ktsmy/aws-lambda-cobol-sample).  I left it at that.

Recently I was triggered to re-visit the project, I forked the repo and made some changes, added a [SAM emplate](https://aws.amazon.com/serverless/sam/) adjusted the build scripts and exposed the result as an invokable http endpoint, you can see the result here:

<iframe width="625" height="30" frameborder="0" scrolling="no" marginheight="0" marginwidth="0" src="https://fe9yjg76ei.execute-api.ap-southeast-2.amazonaws.com/Prod/function1" style="border: 1px solid black;background: #FFFFFF;"></iframe>

[Demo URL - Function 1:](https://fe9yjg76ei.execute-api.ap-southeast-2.amazonaws.com/Prod/function1)

[Demo URL - Function 2:](https://fe9yjg76ei.execute-api.ap-southeast-2.amazonaws.com/Prod/function2)

I found a COBOL [IDE](https://pypi.org/project/OpenCobolIDE/), if I'm going to give it a go I might as-well have as much helper as possible, COBOL only allows max line length of 128 characters, for a while I was deeply confused as to why I have received syntax errors where I have simply made the lines to long.
```
3.1.1 Command Line Length
16-bit:
On the 16-bit COBOL system, the maximum length of command lines is 128 characters. The name of the program being called will take up some of this, leaving a lower limit for the parameters to be passed to the program.

32-bit:
On 32-bit COBOL systems the maximum length of a command line is determined by the operating system.
```

Please take little notice to the code, string concatenation in 3 different ways, 2 function calls and a loop, hey I built the whole example in a few hours included loots of googling "COBOL string manipulation, COBOL loops" etc.  However I did mimic the expected response that API gateway wants to see and was able to use SAM to publish an API, I can also run this locally.  There are 2 shell scripts and a Dockerfile, these are well documented and at a high level this is the flow:

Start a container based of Amazon linux v1, install the dependencies required to compile a COBOL program, copy across the COBOL source files, compile each COBOL program, zip the required artefacts and copy it out of the container ready for deployment.  The handler for each path on the API uses convention to map to a COBOL program of the same name eg:

![COBOL](https://cdn.kukiel.dev/cobol-build.gif "COBOL")

- https://{url}/program1  ->  program1.cob
- https://{url}/programTwo  ->  programTwo.cob

You do not need to COBOL compiler locally to run the tests but you will want one as I made many mistakes while becoming familiar you can install this on MACOS with:

```
brew install gnu-cobol
```

[Other OS CLlick here](https://sourceforge.net/p/open-cobol/wiki/Install%20Guide/)

The IDE
![COBOL](/assets/post/2020-05-22-COBOL-on-aws-lambda/cobol.png "COBOL")

