# [알고리즘] 이진트리 - minimum-depth-of-binary-tree

LeetCode

- minimum-depth-of-binary-tree
- https://leetcode.com/problems/minimum-depth-of-binary-tree/

문제 개요

- 주어진 이진트리에서 가장 짧은 깊이를 구하는 문제
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
const minDepth = function (root) {
  if (!root) return 0;

  const left = minDepth(root.left);
  const right = minDepth(root.right);

  //최소값을 구하므로 최소값을 먼저 구하고, 만약 최소값이 없다면 최대값을 구한다.
  return 1 + (Math.min(left, right) || Math.max(left, right));
};
```
