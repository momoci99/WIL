# [알고리즘] 이진트리 - maximum-depth-of-binary-tree

LeetCode

- maximum-depth-of-binary-tree
- https://leetcode.com/problems/maximum-depth-of-binary-tree/submissions/

문제 개요

- 주어진 이진트리에서 가장 깊은 레빌을 구하는 문제
- 깊이는 루트노드에서 리프노드까지의 거리를 의미.

참고 링크

- https://zereight.tistory.com/841

전개

- 입력되는 객체가 숫자가 아닌 객체이므로 주의
- 재귀적으로 각 노드들을 순회하면서 depth를 계산한다.
- 가장 작은 단위부터 생각해야 쉽게 풀리는 문제.

생각

- LeetCode는 일반적인 문자 배열이 아니라 고유 객체를 입력으로 받는 케이스가 많은 듯.
- 재귀 탐색이라 그런지 공간 복잡도가 높음.

```js
/**
 *
 * 입력으로 주어지는 node 객체는 아래와 같다.
 * function TreeNode(val, left, right) {
 *     this.val = (val===undefined ? 0 : val)
 *     this.left = (left===undefined ? null : left)
 *     this.right = (right===undefined ? null : right)
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number}
 */
const maxDepth = function (root) {
  if (!root) return 0;

  const left = maxDepth(root.left);
  const right = maxDepth(root.right);

  return 1 + (Math.max(left, right) || Math.min(left, right));
};
```
