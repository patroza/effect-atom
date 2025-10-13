<script setup lang="ts">
import { Atom, useAtom, useAtomSet, useAtomValue } from "@effect-atom/atom-vue"
import { onMounted, onUnmounted, ref, watch } from "vue"
import { TestClient } from "../fixtures/TestClient";
import { Effect, Exit } from "effect";

defineProps<{ msg: string }>()

const count = ref(0)

const req = ref({ echo: "Hello World" })

let i = 0

const atom = Atom.fn((req: string) => Effect.gen(function* () {
  yield* Effect.sleep(1_000)
  return { i: i++, d: new Date().toISOString() }
}), { concurrent: true})

// const atom2 = Atom.fn((req: string, get) => Effect.gen(function* () {
//   get.set(atom, req)
// }), { concurrent: true})
// const atom3 = Atom.fn((req: string, get) => Effect.gen(function* () {
//   get.set(atom, req)
// }), { concurrent: true})

const atom4 = Atom.writable(
  (get) => get(atom), // Initial value for the writable
  (get) => {
    // set(countAtom, update); // This would update countAtom directly
    // Or to be safer, you can use a derived atom
    get.set(atom, "new"); // This will wait for the derived atom to update
  }
);

const atom5 = Atom.writable(
  (get) => get(atom), // Initial value for the writable
  (get) => {
    // set(countAtom, update); // This would update countAtom directly
    // Or to be safer, you can use a derived atom
    get.set(atom, "new2"); // This will wait for the derived atom to update
  }
);

const [getAtom, setAtom] = useAtom(() => atom)
const [getAtom4, setAtom4] = useAtom(() => atom4)
const [getAtom5, setAtom5] = useAtom(() => atom5)

watch(getAtom, (newVal) => {
  console.log("Atom changed:", newVal);
});
watch(getAtom4, (newVal) => {
  console.log("Atom4 changed:", newVal);
});
watch(getAtom5, (newVal) => {
  console.log("Atom5 changed:", newVal);
});
onMounted(() => { setAtom("test"); setTimeout(() => setAtom5("test"), 5_000)  })

const result = useAtomValue(() => {
  console.log("Computing Atom:", req.value)
  return Atom.refreshOnWindowFocus(TestClient.query("Get", req.value, { reactivityKeys: ["Get"]}))
})

const set = useAtomSet(() => TestClient.mutation("Set"), { mode: "promiseExit" })

const intervalEnabled = ref(false)

const interval = setInterval(
  () =>
    intervalEnabled.value &&
    (req.value = { echo: `Hello World ${new Date().toLocaleTimeString()}` }),
  5_000,
)
const onSet = () => set({ payload: { state: "state "+ new Date().toISOString() }, reactivityKeys: ["Get"] })
  .then(_ => console.log("finished", Exit.match(_, { onSuccess: _ => ({ success: _ }), onFailure: _ => ({ failure: _})})))
onUnmounted(() => clearInterval(interval))
</script>

<template>
  <h1>{{ msg }}</h1>

  <div v-if="result._tag === 'Initial'">Initial</div>
  <div v-else-if="result._tag === 'Failure'">
    <div v-if="result.waiting">Waiting...</div>
    Failure.. {{ result.cause }}
  </div>
  <div v-else>
    <div v-if="result.waiting">Waiting...</div>
    Success: {{ result.value }}
  </div>

  <button @click="onSet">Set state</button>

  <button @click="intervalEnabled = !intervalEnabled">
    Toggle interval {{ !intervalEnabled ? "ON" : "OFF" }}
  </button>


  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Learn more about IDE Support for Vue in the
    <a
      href="https://vuejs.org/guide/scaling-up/tooling.html#ide-support"
      target="_blank"
      >Vue Docs Scaling up Guide</a
    >.
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
