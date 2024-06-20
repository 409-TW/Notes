# 可能會常用的code

## PSI

### 所有品項庫存 999

```js
const pages = require('../static/PageMapping')
const { getProductStandardUnit } = require("../PSI-tools/tools/product")

const inventoryPage = pageManager.getPageInstance(pages.inventory)
const warehouses = await database.collection(pages.warehouse).aggregate([
  {
    $project: {
      uuid: 1,
      warehouseId: "$data.warehouseId",
      warehouseName: "$data.warehouseName",
    }
  }
]).toArray()
const inventories = await database.collection(pages.inventory).aggregate([
  {
    $project: {
      productId: "$data.productId.value",
      warehouseId: "$data.warehouseId.value",
    }
  }
]).toArray()

const cursor = await database.collection(pages.product).find({})
const datasets = []

while (await cursor.hasNext()) {
  const product = await cursor.next()
  
  if (product.data.productType != "option1") continue

  for (let warehouse of warehouses) {
    const inventory = inventories.find(_ => (
      _.warehouseId == warehouse.warehouseId &&
      _.productId == product.data.productId
    ))

    if (!dataTool.isEmpty(inventory)) {
      await database.collection(pages.inventory).updateOne(
        { _id: inventory._id },
        { $set: { "data.quantity": dataTool.toDecimal128(999) }}
      )
      continue
    }

    const unit = getProductStandardUnit(product)
    const dataset = datasetTool.getDefaultDataset(
      inventoryPage.getMain(),
      {
        productId: {
          value: product.data.productId,
          referenceIDs: { [`${pages.product}$_main$uuid`]: product.uuid }
        },
        productName: product.data.productName,
        unitId: unit.data.unitId,
        unitName: unit.data.unitName,
        warehouseId: {
          value: warehouse.warehouseId,
          referenceIDs: { [`${pages.warehouse}$_main$uuid`]: warehouse.uuid }
        },
        warehouseName: warehouse.warehouseName,
        quantity: dataTool.toDecimal128(999)
      }
    )
    
    await datasetManager.setDatasetBase({ ctx, dataset })

    datasets.push(dataset)
  }      
}

await database.collection(pages.inventory).insertMany(datasets)
```
