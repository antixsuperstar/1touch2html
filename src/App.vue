<script setup lang="ts">
import { ref, reactive, watch } from 'vue'
const idioma = ref('es')
let formateador = new Intl.DateTimeFormat(idioma.value ? [idioma.value] : idioma.value, { month: 'long' })
const ajustarFormateador = () => {
	formateador = new Intl.DateTimeFormat(idioma.value ? [idioma.value] : idioma.value, { month: 'long' })
}
const mesANumero = (mes: string) => {
	const fecha = new Date('2000-01-01T00:00:00.000')
	return Array(12)
		.fill(0)
		.map((i, n) => {
			fecha.setMonth(n)
			return formateador.format(fecha)
		})
		.findIndex(item => !item.localeCompare(mes, idioma.value, { sensitivity: 'base' }))
}
type LangPresets = Record<string, {
	display: string,
	transform: (d: string) => string,
}>
const langpresets: LangPresets = {
	es: {
		display: 'es - Español',
		transform: (datestr) => {
			const busqueda = datestr.match(/(?<day>\d+) de (?<month>[^ ]+) de (?<year>\d+), (?<hour>\d+):(?<minute>\d+)/)
			if (!busqueda || !busqueda.groups) return datestr

			console.dir({ busqueda, datestr })
			return `${busqueda.groups.year}-${mesANumero(busqueda.groups.month)+1}-${busqueda.groups.day} ` +
				`${busqueda.groups.hour}:${busqueda.groups.minute}`
		},
	},
	// en: { display: 'en - English', ... }
	// ...
}
watch(
	idioma,
	() => {
		if (idioma.value && idioma.value.length === 2) {
			ajustarFormateador()
		}
	}
)
ajustarFormateador()


import { computed } from 'vue'
import { parse, type Options } from 'csv-parse/browser/esm/sync'
const csvinput = ref<HTMLInputElement>()
const contenido = reactive<string[][]>([])
const mostrarContenido = ref(false)
const invertirOrden = ref(true)
const datos = reactive<string[][]>([])
const datosOrdenados = computed(() => invertirOrden.value ? Array.from(datos).reverse() : datos)
const leerArchivo = () => {
	if (!csvinput.value?.files || !csvinput.value.files.length) {
		return
	}
	csvinput.value.files[0].text()
		.then(texto => {
			// Interpreto el CSV
			const opciones: Options = {
				skip_empty_lines: true,
				fromLine: 2,
			}
			contenido.splice(0, 0, ...parse(texto, opciones))

			// Transformo valores de fecha (columna 2) a valores más manejables
			datos.splice(0)
			contenido.forEach((linea: string[]) => {
				datos.push([
					langpresets[idioma.value].transform(linea[1]),
					linea[2],
				])
			})
		})
}
</script>

<template>
	<div class="p-2 bg-white dark:bg-gray-800 dark:text-gray-500">
		<ol class="mb-2">
			<li>What <a class="font-bold underline" target="_blank" href="https://en.wikipedia.org/wiki/IETF_language_tag">language</a> is the data CSV in?
				<input type="text"
					   v-model="idioma"
					   list="langpresets"
					   class="w-8 border-b-gray-300 border-2 rounded-2"
				/>
				<datalist id="langpresets">
					<option v-for="(v, k) of langpresets"
							:key="`langpreset-${k}`"
							:value="k">{{ v }}</option>
				</datalist>
			</li>
			<li>
				<label>Load the CSV data file:
					<input type="file" accept="text/csv" ref="csvinput" @change="leerArchivo" />
				</label>
			</li>
		</ol>
		<div class="sticky top-0 z-50 bg-white dark:bg-gray-800">
			<input type="checkbox"
				   class="mr-2"
				   id="mostrarContenido"
				   v-model="mostrarContenido" />
			<label for="mostrarContenido">Show loaded content?</label>
		</div>
		<div class="flex flex-col md:flex-row gap-1">
			<div v-if="mostrarContenido"
				 class="flex-auto">
				<pre>{{ contenido }}</pre>
			</div>
			<div v-if="contenido.length"
				 class="flex-auto">
				<div>
					<input type="checkbox"
						   class="mr-2"
						   id="invertir"
						   v-model="invertirOrden" />
					<label for="invertir">Reverse order?</label>
				</div>
				<table>
					<tr v-for="(dato, n) of datosOrdenados"
						:key="`dato-${n}`">
						<td class="px-1">{{ dato[0] }}</td>
						<td class="px-1">{{ dato[1] }}</td>
					</tr>
				</table>
			</div>
		</div>
	</div>
</template>
