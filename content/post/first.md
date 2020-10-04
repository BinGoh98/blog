---
title: "leetcode 331"
date: 2020-10-04T13:01:27+08:00
draft: false
tags: ["leetcode", "算法"]
categories: ["算法"]
---

331
<!--more-->

```go
var ans [][]string
var tmp []string

func partition(s string) [][]string {
    ans = make([][]string,0)
    tmp = make([]string, 0)

    dfs(s)
    return ans

}

func dfs(s string) {
    // 1. 终止条件
    if len(s) == 0 {
		x := make([]string, len(tmp))
		copy(x, tmp)
		ans = append(ans, x)
		return
	}

    // 2. 递归
    j := 1
    for j <= len(s) {
        if validate(s[:j]) {
            tmp = append(tmp, s[:j])
            dfs(s[j:])
            tmp = tmp[:len(tmp) - 1]
        }
        j++
    }
}

func validate(s string) bool{
    i := 0
    j := len(s) - 1
    for i < j {
        if s[i] == s[j] {
            i++
            j--
        }else{
            return false
        }
    }
    return true
}
```