<template>
<form class="grid gap-4 grid-rows-4 grid-cols-3">
  <div class="col-start-1 row-start-1 flex items-start flex-col" >
    <label class="" for="cantidadAfiliados">Cantidad de Afiliados:</label>
    <input class="w-full" min="0" type="number" id="cantidadAfiliados" v-model="form['cantidadAfiliados']">
  </div>
  <div class="col-start-2 row-start-1 col-end-4 inline-grid gap-2 grid-cols-2 ">
    <label class="" for="cuotaMax cuotaMin">Cuota:</label>
    <!-- <input class="col-start-2" step=0.01 placeholder="min" type="number"  id="cantidadAfiliados" v-model="cuota['cuotaMax']">
    <input class="col-start-2" step=0.01 placeholder="max" type="number" id="cantidadAfiliados" v-model="cuota['cuotaMin']"> -->
      <p class=" col-start-2 text-left bg-slate-700">Maximo:{{cuota['cuotaMax']}}</p>
      <p class="col-start-2 text-left bg-slate-700">Minimo:{{cuota['cuotaMin']}}</p>
  </div>
  <div class="flex items-start flex-col col-start-1 row-start-2">
    <label class="" for="importeTotal">Importe Total:</label>
    <input class="w-full " type="number" id="importeTotal" v-model="form['importeTotal']">
  </div>
  <button class=" col-start-2 row-start-3" @click="loadTable($event)">Aceptar</button>
</form>

  <table class="table-auto rounded w-full">
    <thead>
      <tr>
        <th>Cantidad De Afiliados</th>
        <th>Cuota Maxima</th>
        <th>Cuota Minima</th>
        <th>Importe Total</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="item in tabla">
        <td>{{item.cantidadAfiliados}}</td>
        <td>{{item.cuotaMax}}</td>
        <td>{{item.cuotaMin}}</td>
        <td>{{item.importeTotal}}</td>
      </tr>
    </tbody>
  </table>
</template>
<script setup>
import { ref } from 'vue'

const cuota = ref({
  cuotaMax: null,
  cuotaMin: null,

})
const form = ref({
  cantidadAfiliados:null,
  importeTotal:null
})
const tabla = ref([])
const loadTable = (e) => {
  e.preventDefault()
  console.log(form.value)

  for(let key in form.value[0]){
    if(form.value[key] == null || form.value[key] == undefined || form.value[key] == 0){
      alert('Complete todos los campos')
      return
    }
    if(form.value[key] < 0){
      alert('No se admiten valores negativos')
      return
    }
    if(form.value[key] > 1000000000){
      alert('No se admiten valores tan grandes')
      return
    }
  }
  handleRandomsMinMax()
  // if(cuota.value['cuotaMin'] > cuota.value['cuotaMax']){
  //   alert('La cuota minima no puede ser mayor a la cuota maxima')
  //   return
  // }
  console.log(cuota.value)
  form.value.cuotaMax = cuota.value['cuotaMax']
  form.value.cuotaMin = cuota.value['cuotaMin']
  tabla.value.push(form.value)
  form.value = {
    cantidadAfiliados:null,
    cuotaMax:null,
    cuotaMin:null,
    importeTotal:null
  }
}
const handleRandomsMinMax = () => {
  let min = form.value.importeTotal / form.value.cantidadAfiliados
  let max = min * 1.5
  cuota.value['cuotaMax'] = Math.round(max * 100) / 100
  cuota.value['cuotaMin'] = Math.round(min * 100) / 100
}
</script>

