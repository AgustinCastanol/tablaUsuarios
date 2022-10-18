<template>
  <form class="grid gap-4 grid-rows-4 grid-cols-3">
    <div class="col-start-1 row-start-1 flex items-start flex-col">
      <label class="" for="cantidadAfiliados">Cantidad de Afiliados:</label>
      <input class="w-full" min="0" type="number" id="cantidadAfiliados" v-model="form['cantidadAfiliados']">
    </div>
    <div class="col-start-2 row-start-1 col-end-4 inline-grid gap-2 grid-cols-2 ">
      <label class="" for="cuotaMax cuotaMin">Cuota:</label>
      <input class="col-start-2" step=0.01 placeholder="min" type="number" id="cantidadAfiliados"
        v-model="form['cuotaMax']">
      <input class="col-start-2" step=0.01 placeholder="max" type="number" id="cantidadAfiliados"
        v-model="form['cuotaMin']">
    </div>
    <div class="flex items-start flex-col col-start-1 row-start-2">
      <label class="" for="importeTotal">Importe Total:</label>
      <input class="w-full " type="number" id="importeTotal" v-model="form['importeTotal']">
    </div>
    <button class=" col-start-2 row-start-3" @click="loadTable($event)">Aceptar</button>
  </form>

  <table v-if="tabla.length != 0 " class="table table-auto w-full">
    <thead>
      <tr>
        <th>Afiliado</th>
        <th>Cuota</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in tabla">
        <td>{{item.afiliado}}°</td>
        <td>${{item.cuota}}</td>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <td>Total</td>
        <td>${{totalSum}}</td>
      </tr>
    </tfoot>
  </table>
</template>
<script setup>
import { ref } from 'vue'
const totalSum = ref(0)
const form = ref({
  cantidadAfiliados: null,
  importeTotal: null,
  cuotaMax: null,
  cuotaMin: null,
})
const tabla = ref([])
const loadTable = (e) => {
  e.preventDefault()
  console.log(form.value)

  if (form.value['cuotaMax'] == null || form.value['cuotaMax'] == 0) {
    form.value['cuotaMax'] = form.value['importeTotal']
  }
  if (form.value['cuotaMin'] == null || form.value['cuotaMin'] == 0) {
    form.value['cuotaMin'] = 0
  }
  for (let key in form.value) {
    if (form.value[key] == null || form.value[key] == undefined) {
      alert('Complete todos los campos')
      return
    }
    if (form.value[key] < 0) {
      alert('No se admiten valores negativos')
      return
    }
    if (key == 'cantidadAfiliados' || key == 'importeTotal') {
      if (form.value[key] % 1 != 0) {
        alert('No se admiten valores decimales')
        return
      }
      if (form.value[key] == 0) {
        alert('No se admiten valores nulos')
        return
      }
    }
    if (form.value[key] > 1000000000) {
      alert('No se admiten valores tan grandes')
      return
    }
  }
  if (form.value['cuotaMax'] < form.value['cuotaMin']) {
    alert('La cuota maxima no puede ser menor a la cuota minima')
    return
  }

  handleRandomsMinMax()

  // form.value = {
  //   cantidadAfiliados:null,
  //   cuotaMax:null,
  //   cuotaMin:null,
  //   importeTotal:null
  // }
}
const handleRandomsMinMax = () => {
  /*Create a random number within a range, without the sum of the generated values ​​exceeding the total */
  let total = form.value['importeTotal']
  let sum = 0
  let afiliado = 0
  let max = form.value['cuotaMax']
  let min = form.value['cuotaMin']
  tabla.value = []
  while (afiliado < form.value['cantidadAfiliados']) {
    let random = Math.random() * (max - min + 1) + min
    random = Math.round(random * 100) / 100
    if (sum + random <= total) {
      sum += random
      if (random == 0) {
        tabla.value[afiliado - 1].cuota = tabla.value[afiliado - 1].cuota / 2
        sum -= tabla.value[afiliado - 1].cuota
        tabla.value.push({
          afiliado: afiliado + 1,
          cuota: tabla.value[afiliado - 1].cuota
        })
      } else {
        tabla.value.push({
          afiliado: afiliado + 1,
          cuota: random
        })
      }
      afiliado++
    }
  }
  if (sum < total) {
    console.log(sum, "sum")
    let acum = Math.round(((total - sum)) * 100) / 100
    if (acum < 1) {
      tabla.value[tabla.value.length - 1].cuota = Math.round((tabla.value[tabla.value.length - 1].cuota + acum) * 100) / 100 
    } else {
      for (let i = 0; i < afiliado; i++) {
        tabla.value[i].cuota = Math.round((tabla.value[i].cuota + (acum / afiliado)) * 100) / 100 
      }
    }
    sum = total
  }
  totalSum.value = sum


}

</script>

