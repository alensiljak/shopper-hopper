<template>
  <q-page class="q-ma-sm">
    <div class="container">up bottom</div>

    Shopping List

    <q-list>
      <q-item v-for="item in items" :key="item.id">
        <q-item-section>
          {{ products[item.productid].name }}
        </q-item-section>
        <q-item-section side>
          <q-icon name="check_box_outline_blank" />
          <q-icon name="check_box_outline" />
        </q-item-section>
      </q-item>
    </q-list>

    <q-page-sticky position="bottom-right" :offset="[18, 18]">
      <q-btn fab icon="add" color="accent" @click="onFabClick" />
    </q-page-sticky>
  </q-page>
</template>

<script setup lang="ts">
import { onMounted, ref, toRaw } from 'vue'
import { useRouter } from 'vue-router'
import { useMainStore } from 'src/stores/mainStore'
import { ShoppingListItem } from 'src/stores/model'
import { db } from '../stores/persistentStorage'

const router = useRouter()
const store = useMainStore()

const items = ref([])
const products = ref({})

onMounted(async () => {
  // Check if we are back from the selection mode.
  await handleSelection()

  // load data
  await loadData()
})

async function loadData() {
  //
  let listItems = await db.shoppingList.toArray()
  console.log('items:', listItems)
  let productIds = listItems.map((item) => item.productid)

  // load products
  let productRecords = await db.products.bulkGet(productIds)
  console.log('products:', products)

  let productsObject = {}
  productRecords.forEach((product) => (productsObject[product.id] = product))
  products.value = productsObject

  items.value = listItems
}

async function handleSelection() {
  // get the selected product from the state store
  if (!store.selectedProduct) return

  //let productId: number = toRaw(store.selectProduct)
  let productId: number = store.selectedProduct
  // console.log('selected product', store.selectedProduct, 'raw:', productId)
  // add the product to the shopping list
  var item = new ShoppingListItem(productId)
  // var item = db.shoppingList.newRecord(productId)
  //console.log(item instanceof ShoppingListItem)

  await db.shoppingList.add(item)

  // clear the state
  store.selectProduct(null)
}

function onFabClick() {
  router.push({ name: 'Product List', params: { selection: true } })
}
</script>
  
<style lang="scss" scoped>
.container {
  display: flex;
}
</style>