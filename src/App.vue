<template>
  <form class="grid gap-4 grid-rows-4 grid-cols-3">
    <div class="col-start-1 row-start-1 flex items-start flex-col">
      <label class="" for="cantidadAfiliados">Cantidad de Afiliados:</label>
      <input class="w-full" min="0" type="number" id="cantidadAfiliados" v-model="form['cantidadAfiliados']">
    </div>
    <div class="col-start-2 row-start-1 col-end-4 inline-grid gap-2 grid-cols-2 ">
      <label class="" for="cuotaMax cuotaMin">Cuota:</label>
      <input class="col-start-2" step=0.01 placeholder="Max" type="number" id="cantidadAfiliados"
        v-model="form['cuotaMax']">
      <input class="col-start-2" step=0.01 placeholder="Min" type="number" id="cantidadAfiliados"
        v-model="form['cuotaMin']">
    </div>
    <div class="flex items-start flex-col col-start-1 row-start-2">
      <label class="" for="importeTotal">Importe Total:</label>
      <input class="w-full " type="number" id="importeTotal" v-model="form['importeTotal']">
    </div>
    <button :disable="loading" class=" col-start-1 row-start-3" @click="loadTable($event) "
      :style="loading?'opacity:0.4;':''">Aceptar</button>
    <button :disable="loading" class="col-start-3 row-start-3" @click="refreshTable($event)"
      :style="loading?'opacity:0.4;':''">Refresh</button>
  </form>
  <div v-if="loading">
    <div class="">Cargando afiliados.....
    </div>
    <div class="lds-dual-ring"></div>
  </div>
  <div v-if="tabla.length > 0 && !loading "
    class="table mx-auto w-2/3 flex items-center border-solid border-2 border-neutral-50">
    <div class="table-header-group">
      <div class="table-row">
        <div class="table-cell border-solid border-r border-neutral-50 text-left">
          <span>Afiliado</span>
        </div>
        <div class="table-cell text-left">
          <span>Cuota</span>
        </div>
      </div>
    </div>
    <div class="table-row-group">
      <div class="table-row my-2" v-for="item in tabla">
        <div class="table-cell border-solid border-t border-r border-neutral-50 text-left">
          <span>°{{item.afiliado}}</span>
        </div>
        <div class="table-cell border-solid border-t border-r border-neutral-50 text-left">
          <span>${{item.cuota}}</span>
        </div>
      </div>
      <div class="table-row my-2">
        <div class="table-cell border-solid border-t border-r border-neutrak-59 text-left">
          <span>Total</span>
        </div>
        <div class="table-cell border-solid border-t border-r border-neutral-50 text-left">
          <span>${{totalSum}}</span>
        </div>
      </div>
    </div>
  </div>
  <button v-if="tabla.length > 0" class="float-download" @click="arrayToCsv()">
    Descargar
  </button>
</template>
<script setup>
import { ref } from 'vue'
const totalSum = ref(0)
const afiliadoCargado = ref(0)
const loading = ref(false)
const form = ref({
  cantidadAfiliados: null,
  importeTotal: null,
  cuotaMax: null,
  cuotaMin: null,
})

const tabla = ref([])
const loadTable = async (e) => {
  try {
    e.preventDefault()
    loading.value = true;
    tabla.value = []
    afiliadoCargado.value = 0
    if (form.value['cuotaMax'] == null || form.value['cuotaMax'] == 0) {
      form.value['cuotaMax'] = form.value['importeTotal']
    }
    if (form.value['cuotaMin'] == null || form.value['cuotaMin'] == 0) {
      form.value['cuotaMin'] = 0
    }
    for (let key in form.value) {
      if (form.value[key] == null || form.value[key] == undefined) {
        alert('Complete todos los campos')
        loading.value = false;
        return
      }
      if (form.value[key] < 0) {
        alert('No se admiten valores negativos')
        loading.value = false;
        return
      }
      if (key == 'cantidadAfiliados' || key == 'importeTotal') {
        if (form.value[key] % 1 != 0) {
          alert('No se admiten valores decimales')
          loading.value = false;
          return
        }
        if (form.value[key] == 0) {
          alert('No se admiten valores nulos')
          loading.value = false;
          return
        }
      }
      if (form.value[key] > 1000000000) {
        alert('No se admiten valores tan grandes')
        loading.value = false;
        return
      }
    }
    if (form.value['cuotaMax'] < form.value['cuotaMin']) {
      alert('La cuota maxima no puede ser menor a la cuota minima')
      loading.value = false;
      return
    }

    await handleRandomsMinMaxPromise()

    loading.value = false;

  } catch (error) {
    alert('Error al crear la tabla')
    loading.value = false;
    return
  }

}
const handleRandomsMinMax = () => {
  let total = form.value['importeTotal']
  let sum = 0
  let afiliado = 0
  let max = form.value['cuotaMax']
  let min = form.value['cuotaMin']
  tabla.value = []
  while (afiliado < form.value['cantidadAfiliados']) {
    let random = Math.random() * (max - min + 1) + min
    afiliadoCargado.value = afiliado + 1
      sum += random
      tabla.value.push({
        afiliado: afiliado + 1,
        cuota: Math.round(random * 100) / 100
      })
      afiliado++
  }

  if(sum > total){
    let control = 0
    let next = 0
    let diff = (sum - total)
    let part = Math.round((diff/ (form.value['cantidadAfiliados']))*100)/100
    tabla.value.map((item)=>{
      if(item.cuota > part+next){
        item.cuota = Math.round((item.cuota-(part + next)) * 100) / 100
        next = 0;
      }
      else{
        item.cuota -=  part
        if(item.cuota <0){
          let random = Math.random() * (1000 + 1)
          next = (-1*(item.cuota-random))
          item.cuota = random
        }
      }
      control += item.cuota 
    })
    console.log(control,"control")
  }
  if(sum < total){
    let diff = (total - sum)/form.value['cantidadAfiliados']
    tabla.value.map((item)=>{
      item.cuota = Math.round((item.cuota +diff) * 100) / 100
    })
  }
 totalSum.value =total

}

const refreshTable = (e) => {
  e.preventDefault()
  tabla.value = []
  form.value = {
    cantidadAfiliados: null,
    cuotaMax: null,
    cuotaMin: null,
    importeTotal: null
  }
  afiliadoCargado.value = 0
}

const arrayToCsv = () => {
  let csvContent = "data:text/csv;charset=utf-8,";
  /*how to conver a array of objects in array */
  let array = tabla.value.map((item) => {
    return [item.afiliado, item.cuota]
  })
  array.unshift(['Afiliado', 'Cuota'])
  array.forEach(function (rowArray) {
    let row = rowArray.join(";");
    csvContent += row + "\r\n";
  });
  var encodedUri = encodeURI(csvContent);
  var link = document.createElement("a");
  link.setAttribute("href", encodedUri);
  link.setAttribute("download", "my_data.csv");
  document.body.appendChild(link); // Required for FF
  var encodedUri = encodeURI(csvContent);
  window.open(encodedUri);
}

const handleRandomsMinMaxPromise = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      handleRandomsMinMax()
      resolve('ok')
    }, 10)
  })
}

</script>

