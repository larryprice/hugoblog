---
layout: post
title: "Copy Data Using qmake"
date: "2013-11-14T21:24:00-05:00"
comments: true
categories: [qt, qmake, c++]
---

I'll take things that should be really easy but are in fact impossible for $1000, Mr Trebek.

I have a [Qt](https://qt-project.org/wiki/Qt_5.0) desktop app that uses [qmake](https://qt-project.org/doc/qt-4.8/qmake-manual.html). I have tests. I have test data that I need to copy to my build directory. Not only that, but I want to copy a whole directory recursively. In Linux _and_ Windows. You must be thinking to yourself, 'There's no way that's possible, leave me out of your crazy plans!' Well to that I say nay. Nay indeed.

I originally copied my test data using the `INSTALLS` variable. My data folder is just a directory sitting in the test source called "TestData." This solution is fantastic because it works cross-platform.

{{< code_block syntax="make" description="MyBAProjectTest.pro" >}}
install_it.path = $${OUT_PWD}
install_it.files += TestData/

INSTALLS += install_it
{{< /code_block >}}

However, this means that if I ever have a clean directory, I have to run `make install` to install my test data. What a pain! Why bother! Why even write tests when life is this unjust!

BUT WAIT! After combing the depths of the internets for much of the evening, I was able to find many halfway solutions to my original problem. The concept is to add commands to the `QMAKE_POST_LINK` variable.

For Linux, I use `cp`:

{{< code_block syntax="make" description="MyBAProjectTest.pro" >}}
QMAKE_POST_LINK += $$quote(cp -rf $${PWD}/TestData $${OUT_PWD})
{{< /code_block >}}

For Windows, I use `xcopy`:

{{< code_block syntax="make" description="MyBAProjectTest.pro" >}}
PWD_WIN = $${PWD}
PWD_WIN ~= s,/,\\,g

QMAKE_POST_LINK += $$quote(mkdir DestFolder)
QMAKE_POST_LINK += $$quote(xcopy $${PWD_WIN}\\TestData $${OUT_PWD_WIN}\\TestData /E)
{{< /code_block >}}

I also want that directory to be deleted when I run `make clean`. Cleaning up just means adding some commands to the `QMAKE_CLEAN` directive. I also want to run the appropriate commands whether I'm on Unix or Windows without having to modify my `.pro` file. Wrapping the previous commands in what I call an "OS-space" will cause those commands to only run in the specified operating system.

{{< code_block syntax="make" description="MyBAProjectTest.pro" >}}
win32 {
    PWD_WIN = $${PWD}
    PWD_WIN ~= s,/,\\,g

    QMAKE_POST_LINK += $$quote(mkdir DestFolder)
    QMAKE_POST_LINK += $$quote(xcopy $${PWD_WIN}\\TestData $${OUT_PWD_WIN}\\TestData /E)

    QMAKE_CLEAN += /s /f /q TestData && rd /s /q TestData
}

unix {
    QMAKE_POST_LINK += $$quote(cp -rf $${PWD}/TestData $${OUT_PWD})

    QMAKE_CLEAN += -r TestData
}
{{< /code_block >}}

Data copied, tests working again. Take that, Nokia; no matter how difficult you make your build-processing tool, I'll figure out how to contort it to my whims.
