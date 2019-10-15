---
author: robertwang
layout: post
title:  "RxJava 及 Kotlin Coroutine 的混用"
date:   2019-10-10
categories: rxjava kotlin coroutine

---

由於目前的專案從 RxJava 1.x 開始使用，經過三個版本的開發後，目前已經用到了 RxJava 2.x，因此，大部分底層共用的 Manager/Handler 類的實作都是以 RxJava 的達成。直到....我們開始嘗試使用 Coroutine！

想像，當你在 MVP 的 Presenter 或 MVVM 的 ViewModel 裡開始用起了 Coroutine，卻得呼叫底層是以 RxJava 來實作的 function，這時候還接得起來嗎？（謎之音：當然有接起來，不然來敢來這邊寫...）

