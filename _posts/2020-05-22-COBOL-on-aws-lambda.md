---
layout: post
title:  "Runnign Cobol on AWS Lambda"
date:   2020-05-22 018:18:12 +1000
categories: aws COBOL lambda ec2 costsavings docker
---
![COBOL](https://cdn.kukiel.dev/cobol-book.jpg =300)


I have spent part of my workign carrer in Financial services, it's a great industry and an anyone who has spent any time there knows Mainframe, COBOL and Fortran do come up.  I know 1 of one COBOL programmer personally ( the wife of my friend ), how ever have never see COBOL code myself unitll ~a year ago where I did some research, found a compiler and tried it.  I then did a quick google about running this in Lambda ( why not? ) and found this repo:

Recently I was triggered to re-visit the project, I forked the repo and made some changes, added SAM adjusted the build scripts and exposed the resutl as a invokabe http endpoint, you can see the result here:

[Demo URL - Function 1:](https://fe9yjg76ei.execute-api.ap-southeast-2.amazonaws.com/Prod/function1)

[Demo URL - Function 2:](https://fe9yjg76ei.execute-api.ap-southeast-2.amazonaws.com/Prod/function2)

I found a COBOL IDE, if I'm goign to give it a go I might aswell have a helper, COBOL only allows max line length of 128 chararcters, for a while I was deeply confused as to why I have recieve syntax errors where I have simply made the lines to long.
```
3.1.1 Command Line Length
16-bit:
On the 16-bit COBOL system, the maximum length of command lines is 128 characters. The name of the program being called will take up some of this,leaving a lower limit for the parameters to be passed to the program.

32-bit:
On 32-bit COBOL systems the maximum length of a command line is determined by the operating system.
```

Please take little notice to the code, string concatination in 3 differnt ways, hey I built the whoel example in a few hours included loots of googling "cobol string manipulation, cobol loops" etc.  However I did mimic the expected response that API gateway wants to see and was able to use SAM to publish an API, I can also run this locally.  There are 2 shell scripts and a Dockerfile, these are well documented and at a high level this is the flow:

Start a container based of Amazon linix v1, install the dependancies required to compile a cobol app, copy across the COBOL source files, compile each COBOL program, zip the required artifacts and copy it out of the container ready for deployment.  The handler for each path on the API uses convention to map to a COBOL program of the same name eg:

- https://{url}/program1  ->  program1.cob
- https://{url}/programTwo  ->  programTwo.cob

You do not need to COBOL compiler locally to run the tests but you will want one as I made meany mistakes while becoming familiur you can install this on MACOS with:

```
brew install gnu-cobol
```

[Other OS CLick here](https://sourceforge.net/p/open-cobol/wiki/Install%20Guide/)

The IDE
![COBOL](/assets/post/2020-05-22-COBOL-on-aws-lambda/cobol.png "COBOL")

