---
title: spaceemacs 学习笔记
date: 2016-11-01 13:45:31
tags: 
  - emacs
  - spcaemacs
  - note
---
   1. gg moves to the beginning of the buffer.
      G  moves to the end of the buffer.
      :  followed by a line number the <enter> moves to that line number .
      
  2. typing / followed by a pharse searches FORWARD for the pharse.
     typing ? followed by a pharse searches BACKWORD for the pharse.
     After a search type n to find the next occurrence in the same direction
     or N to search in the opposite direction.
     
  3. typing % while the cursor is on a(,),[,],{,or} locates its matching pair.
  
  4. To substitute new for the first old on a line type :s/old/new
     To substitute new for all 'old's on a line type    :s/old/new/g
     To substitute phrases between two line #'s type    :#,#s/old/new/g
     To substitute all occurrences in the file type     :%s/old/new/g
     To ask for confirmation each time add 'c'          :%s/old/new/gc
     
  5. To save part of a file type  :#,# w filename
