title: Clojure 每日练习
date: 2015-06-29 19:59:42
tags: Clojure
categories: 记录笔记
description: 函数式的启迪，小括号的逆袭。
---

最近公司的 iOS 项目进入 Debug 阶段，有些沉闷。于是乎从今天开始，尝试每天做一些 Clojure 的练习，找找刺激。[4clojure](http://www.4clojure.com/) 是个不错的网站，上面有很多 Clojure 的小题目，可以学习并巩固 Clojure 的知识。在每一题后面都附上了一段代码，包含：注释了的题目、我的答案和其他人的参考答案，欢迎一起讨论~


### [#001: Nothing but the Truth](http://www.4clojure.com/problem/1)

    ;; This is a clojure form.
    ;; Enter a value which will make the form evaluate to true.
    ;; Don't over think it!
    ;; Hint: true is equal to true.
    ;; (= __ true)

    (= (+ 1 1) 2)

    (not false)

### [#002: Simple Math](http://www.4clojure.com/problem/2)

    ;; If you are not familiar with polish notation
    ;; , simple arithmetic might seem confusing.
    ;; Note: Enter only enough to fill in the blank
    ;; In this case, a single number.
    ;; (= (- 10 (* 2 3)) __)

    4

### [#003: Intro to Strings](http://www.4clojure.com/problem/3)

    ;; Clojure strings are Java strings.  
    ;; This means that you can use any of the Java string methods on Clojure strings.
    ;; (= __ (.toUpperCase "hello world"))

    "HELLO WORLD"

    (reify Object (equals [_ _] (= _))) ;; 花样炫技



### [#004: Intro to Lists](http://www.4clojure.com/problem/4)

    ;; Lists can be constructed with either a function or a quoted form.
    ;; (= (list __) '(:a :b :c))

    :a :b :c



### [#005: Lists: conj](http://www.4clojure.com/problem/5)

    ;; When operating on a list, the conj function will return
    ;; a new list with one or more items "added" to the front.
    ;; (= __ (conj '(2 3 4) 1))

    '(1 2 3 4)

### [#006: Intro to Vectors](http://www.4clojure.com/problem/6)

    ;; Vectors can be constructed several ways.  You can compare them with lists.
    ;; (= [__] (list :a :b :c) (vec '(:a :b :c)) (vector :a :b :c))

    :a :b :c

### [#007: Vectors: conj](http://www.4clojure.com/problem/7)

    ;; When operating on a Vector, the conj function will return a new vector with one or
    ;; more items "added" to the end.
    ;; (= __ (conj [1 2 3] 4))

    [1 2 3 4]


### [#008: Intro to Sets](http://www.4clojure.com/problem/8)

    ;; Sets are collections of unique values.
    ;; (= __ (set '(:a :a :b :c :c :c :c :d :d)))

    #{:a :b :c :d}



### [#009: Sets: conj](http://www.4clojure.com/problem/9)

    ;; When operating on a set, the conj function returns a new set with one or more keys
    ;; "added".
    ;; (= #{1 2 3 4} (conj #{1 4 3} __))

    2

### [#010: Intro to Maps](http://www.4clojure.com/problem/10)

    ;; Maps store key-value pairs.  Both maps and keywords can be used as lookup functions.
    ;; Commas can be used to make maps more readable, but they are not required.
    ;; (= __ ((hash-map :a 10, :b 20, :c 30) :b))

    20

### [#011: Maps: conj](http://www.4clojure.com/problem/11)

    ;; When operating on a map, the conj function returns a new map with one or more
    ;; key-value pairs "added".
    ;; (= {:a 1, :b 2, :c 3} (conj {:a 1} __ [:c 3]))

    {:b 2}

    [:b 2]

### [#012: Intro to Sequences](http://www.4clojure.com/problem/12)

    ;; All Clojure collections support sequencing.  You can operate on sequences with
    ;; functions like first, second, and last.
    ;; (= __ (first '(3 2 1)))

    3

### [#013: Intro to Sequences](http://www.4clojure.com/problem/13)

    ;; The rest function will return all the items of a sequence except the first.
    ;; (= __ (rest [10 20 30 40]))
    (= [20 30 40] (rest [10 20 30 40]))

    [20 30 40]


### [#014: Intro to Functions](http://www.4clojure.com/problem/14)

    ;; Clojure has many different ways to create functions.
    ;; (= __ ((fn add-five [x] (+ x 5)) 3))

    8

### [#015: Double Down](http://www.4clojure.com/problem/15)

    ;; Write a function which doubles a number.
    ;; (= (__ 2) 4)

    #(* 2 %)

    * 2
    (fn [x] (* x 2))
    (partial * 2)
    #(+ % %)

### [#016: Hello World](http://www.4clojure.com/problem/16)

    ;; Write a function which returns a personalized greeting.
    ;; (= (__ "Dave") "Hello, Dave!")

    #(str "Hello, " % "!")

    (fn [n] (str "Hello, " n "!"))
    #(str "Hello, " % \!)
