# bktree
BK-Tree for autocorrect


```js
function zeros(n) {
  return Array(n).fill(0)
}

function matrix(row, col) {
  return Array(row).fill(
    () => zeros(col)
  ).map(fn => fn())
}

// Other edit distance includes hamming, jaccard etc.
function editDistance(s, t) {
  const [m, n] = [s.length, t.length]
  const dp = matrix(m + 1, n + 1)
  for (let i = 0; i <= m; i += 1) {
    dp[i][0] = i
  }
  for (let j = 0; j <= n; j += 1) {
    dp[0][j] = j
  }
  for (let i = 1; i <= m; i += 1) {
    for (let j = 1; j <= n; j += 1) {
      if (s[i - 1] !== t[j - 1]) {
        dp[i][j] = Math.min(
          1 + dp[i - 1][j], // Deletion.
          1 + dp[i][j - 1], // Insertion.
          1 + dp[i - 1][j - 1] // Replacement.
        )
      } else {
        dp[i][j] = dp[i - 1][j - 1]
      }
    }
  }
  return dp[m][n]
}

class Node {
  constructor(word = '') {
    this.word = word
    this.children = {}
  }
}

class BKTree {
  static TOL = 2

  constructor(dictionary = [], distanceFunction = editDistance) {
    this.distanceFunction = distanceFunction
    for (let word of dictionary) {
      this.add(word)
    }
  }

  add(str) {
    if (!this.root) {
      this.root = new Node(str)
      return
    }
    let curr = this.root
    let dist = this.distanceFunction(str, curr.word)
    while (curr.children[dist]) {
      curr = curr.children[dist]
      dist = this.distanceFunction(str, curr.word)
    }
    curr.children[dist] = new Node(str)
  }

  query(str) {
    const result = []
    const candidates = [this.root]

    while (candidates.length) {
      const node = candidates.pop()
      if (!node.word) {
        return result
      }
      const dist = this.distanceFunction(node.word, str)
      if (dist <= BKTree.TOL) result.push(node.word)

      let start = dist - BKTree.TOL
      if (start < 0) start = 1
      for (let i = start; i < dist + BKTree.TOL; i += 1) {
        if (node.children[i]) {
          candidates.push(node.children[i])
        }
      }
    }
    return result
  }
}

const dictionary = ['hell', 'help', 'shel', 'smell', 'fell', 'felt', 'oops', 'pop', 'oouch', 'halt']

// TODO: Load from generator instead of array.
const bkTree = new BKTree(dictionary)

console.log(bkTree.query('ops'))
console.log(bkTree.query('helt'))
console.log(bkTree.query('shelp'))

console.log(bkTree)
```
