# Breakpad Usage on Embedded Linux

## Objective

This article aims to use Breadpad on Embedded Linux.
After a lot of research on "crash backtrace", I think BreakPad/CrashPad is the best solution for embedded Linux. 
1. BreakPad can rebuild backtrace even the program is un-symbolized on Borad. This is very important as flash size is limited on Embedded system.
2. BreakPad integrate the upload stream which can send the crash information to remote HTTP server.
3. BreakPad is more considered on the correctly information of crash. For example, it clone a new task to dump crash information, resize the stacksize of signal handlers, etc.

Tips: Breakpad need C++14 supported !!!

## Compile for embedded board

Commands:

1. ./configure CC=arm-linux-gnueabihf-gcc CXX=arm-linux-gnueabihf-g++ --host=arm-linux-gnueabihf --prefix=/< your directory >/out --disable-tools --disable-processor

2. make

3. make install

Tips: Disable tools and processor because we don't need to run symbol dump and anaysis crash information on Embedded board.

Output : libbreakpad_client.a

## Compile on Linux(Ubuntu) Host

Commands:
1. ./configure --prefix=/< your directory >/host-out
2. make
3. make install

It will generate a lot tools on Host. Like dump_syms,minidump_stackwalk.

## Intergrate into Embedded Program

Usage: integrate "libbreakpad_client.a" into your program.

arm-linux-gnueabihf-g++ -o breakpad_test breakpad_test.c -g -Wall -O2 -I ./include/breakpad -L./lib/ -lbreakpad_client -lpthread -lrt


./host-out/dump_syms breakpad_test > breakpad_test.sym

head -n1 breakpad_test.sym 
  (MODULE Linux arm A91B2484BE93F98B6004676B57EA1A860 breakpad_test)

mkdir  symbols/breakpad_test/A91B2484BE93F98B6004676B57EA1A860 -p

mv breakpad_test.sym symbols/breakpad_test/A91B2484BE93F98B6004676B57EA1A860

arm-linux-gnueabihf-strip breakpad_test

## Anaysis

upload breakpad_test to board, and run.

Breakpad will generate a crash file like "7c038a7e-0cb1-4db0-b7f6a5b4-c9a942e2.dmp", upload it to host PC.

./host-out/minidump_stackwalk 7c038a7e-0cb1-4db0-b7f6a5b4-c9a942e2.dmp ./symbols/