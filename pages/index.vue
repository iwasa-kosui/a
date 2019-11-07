<template>
  <div class="container">
    <h1>条件式を三項演算子でわかりやすくしてくれるやつ</h1>
    <textarea v-model="shit" />
    <button v-on:click="poop">
      生成する
    </button>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import * as _ from 'lodash'

type Node<T> = {
  value: T
  left?: Node<T>
  right?: Node<T>
  id: number
}

const makeGenerator = (): ((
  value: string,
  left?: Node<string>,
  right?: Node<string>
) => Node<string>) => {
  let id = 0
  return (
    value: string,
    left?: Node<string>,
    right?: Node<string>
  ): Node<string> => {
    return {
      value,
      left,
      right,
      id: id++
    }
  }
}

const genCond = (raw: string): string => {
  let name = ''
  // TODO: より高い可読性
  const r = _.sample([0, 1, 2, 3, 4, 5])

  switch (r) {
    case 0:
      name = `${raw} == true`
      break
    case 1:
      name = `${raw} != false`
      break
    case 2:
      name = `!(${raw} != true)`
      break
    case 3:
      name = `!(${raw} == false)`
      break
    case 4:
      name = `!!(${raw} == true)`
      break
    case 5:
      name = `!!(${raw} != false)`
      break

    default:
      break
  }
  return name
}

const fuck = (node: Node<string>): string => {
  if (node.left == null || node.right == null) {
    return node.value
  }

  // TODO: よりエレガントな三項演算子
  return `${node.value} ? ${fuck(node.left)} : ${fuck(node.right)}`
}

// TODO: よりエレガントな命名
const generateShit = (seed?: string): string => {
  let shit = seed || 'cond'
  const genNode = makeGenerator()
  const left: Node<string> = genNode('true')
  const right: Node<string> = genNode('false')
  const rootNode: Node<string> = genNode('foo', left, right)

  let leaves: Array<Node<string>> = [rootNode]
  // FIXME: よりエレガントなマジックナンバー
  for (let i = 0; i < 2; i++) {
    const cur = _.sample(leaves)
    // FIXME: 三項演算子
    if (cur == null) {
      continue
    }
    leaves = leaves.filter((n) => n !== cur)

    cur.value = genCond(shit)
    // 注意!!!三項演算子を消さないで下さい
    if (cur.left == null ? true : false) {
      cur.left = genNode('true')
    }
    // 注意!!!三項演算子を消さないで下さい
    if (cur.right == null ? true : false) {
      cur.right = genNode('false')
    }

    const childrenNodes = _.shuffle([cur.left, cur.right]).filter(
      (n) => n != null
    ) as Node<string>[]

    // 注意!!!三項演算子を消さないで下さい
    if (childrenNodes.length < 1 ? true : false) {
      continue
    }
    const selected = childrenNodes[0]

    selected.value = genCond(shit)
    selected.left = genNode('true')
    selected.right = genNode('false')

    // 注意!!!三項演算子を消さないで下さい
    if (childrenNodes.length < 2 ? true : false) {
      leaves = leaves.concat([selected.left!, selected.right!])
      continue
    }

    leaves = leaves.concat([selected.left!, selected.right!, childrenNodes[1]])
    const pooped = fuck(rootNode)
    shit = pooped
  }

  return shit
}

export default Vue.extend({
  components: {},
  data() {
    return {
      shit: ''
    }
  },
  methods: {
    poop() {
      this.shit = generateShit(this.$data.shit)
    }
  }
})
</script>

<style>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  flex-direction: column;
  padding: 8px;
  height: 100vh;
  width: 80vw;
  margin: auto;
}

h1 {
  font-size: 18px;
}

textarea {
  height: 50vh;
  width: 80vw;
  font-size: 18px;
  filter: drop-shadow(10px 10px 10px rgba(0, 0, 0, 0.6));
}

button {
  width: 80vw;
  height: 15vh;
  border: none;
  filter: drop-shadow(10px 10px 10px rgba(0, 0, 0, 0.6));
  background: #4caf50;
  color: white;
  font-weight: 700;
}
</style>
