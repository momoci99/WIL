# 제목

링크:https://school.programmers.co.kr/learn/courses/30/lessons/12916

문제 개요

- 문자열 내 p와 y의 개수를 비교하여 같으면 true, 다르면 false를 반환

전개

```js
function solution(s) {
  let pCnt = 0;
  let yCnt = 0;

  s.split("").forEach((char) => {
    const lowerChar = char.toLowerCase();

    if (lowerChar === "p") pCnt++;
    if (lowerChar === "y") yCnt++;
  });

  return pCnt === yCnt;
}
```
