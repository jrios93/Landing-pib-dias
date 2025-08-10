<script setup>
import { ref, onMounted, computed, onUnmounted, watch } from 'vue'
import { 
  Menu, X, MapPin, Calendar, Clock, Users, MessageCircle, Globe, ChevronDown, 
  BookOpen, Quote, Church, CheckCircle, Heart, Utensils, Mic, AlertCircle, 
  Loader, Send, Home, Plus, Minus, Building, Star, Phone, Car, Coffee, 
  Shield, Wifi
} from 'lucide-vue-next'
import bgEvent from '/images/conference2.png'
import logo from '/images/logo/logo-web.webp'
import logo2 from '/images/logo/prueba.webp'

import {scheduleData,pastores,dayLabels,months}from './data'



const tipoUsuario = ref('')
const formData = ref({
  nombre: '',
  apellido: '',
  telefono: '',
  lugarProcedencia: '',
  funcion: '',
  edad: '',
  sexo: '',
  iglesia: '',
  nombrePastor: '',
  fechaLlegadaDay: '',
  fechaLlegadaMonth: '',
  fechaLlegadaYear: '',
  fechaRegresoDay: '',
  fechaRegresoMonth: '',
  fechaRegresoYear: '',
  agenciaViaje: '',
  nombreEsposa: '',
  edadEsposa: '',
  nombreEsposoEsposa: '',
  edadEsposoEsposa: '',
  totalAcompanantes: 0,
  tipoAcompanantes: '',
  necesidadHospedaje: false,
  requerimientosEspeciales: ''
})

const formErrors = ref({})
const isSubmitting = ref(false)
const submitMessage = ref('')
const mobileMenuOpen = ref(false)
const activeDay = ref('jueves')
const expandedSchedule = ref(null)
const showEsposaFields = ref(false)
const showMemberSpouseFields = ref(false)

const eventDate = new Date('2025-09-04T10:00:00')
const timeLeft = ref({ días: 0, horas: 0, minutos: 0, segundos: 0 })
let countdownInterval = null

const updateCountdown = () => {
  const now = new Date()
  const diff = eventDate.getTime() - now.getTime()
  
  if (diff > 0) {
    timeLeft.value = {
      días: Math.floor(diff / (1000 * 60 * 60 * 24)),
      horas: Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
      minutos: Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60)),
      segundos: Math.floor((diff % (1000 * 60)) / 1000)
    }
  } else {
    timeLeft.value = { días: 0, horas: 0, minutos: 0, segundos: 0 }
  }
}

const totalAcompanantesCalculado = computed(() => {
  let total = parseInt(formData.value.totalAcompanantes) || 0
  if (tipoUsuario.value === 'pastor' && showEsposaFields.value) {
    total += 1
  }
  if (tipoUsuario.value === 'miembro' && showMemberSpouseFields.value) {
    total += 1
  }
  return total
})

const spouseLabel = computed(() => {
  if (formData.value.sexo === 'masculino') {
    return 'Esposa'
  } else if (formData.value.sexo === 'femenino') {
    return 'Esposo'
  }
  return 'Cónyuge'
})

const validateForm = () => {
  const errors = {}
  
  if (!tipoUsuario.value) {
    errors.tipoUsuario = 'Debe seleccionar el tipo de registro'
  }
  
  if (!formData.value.nombre.trim()) {
    errors.nombre = 'Los nombres son requeridos'
  } else if (formData.value.nombre.trim().length < 2) {
    errors.nombre = 'Los nombres deben tener al menos 2 caracteres'
  } else if (!/^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]+$/.test(formData.value.nombre.trim())) {
    errors.nombre = 'Los nombres solo pueden contener letras y espacios'
  }
  
  if (!formData.value.apellido.trim()) {
    errors.apellido = 'Los apellidos son requeridos'
  } else if (formData.value.apellido.trim().length < 2) {
    errors.apellido = 'Los apellidos deben tener al menos 2 caracteres'
  } else if (!/^[a-zA-ZáéíóúÁÉÍÓÚñÑ\s]+$/.test(formData.value.apellido.trim())) {
    errors.apellido = 'Los apellidos solo pueden contener letras y espacios'
  }
  
  const phoneRegex = /^(\+?51\s?)?9\d{8}$/
  if (!formData.value.telefono.trim()) {
    errors.telefono = 'El número de celular es requerido'
  } else if (!phoneRegex.test(formData.value.telefono.replace(/\s/g, ''))) {
    errors.telefono = 'Formato válido: 999999999 o +51 999999999'
  }
  
  if (!formData.value.lugarProcedencia.trim()) {
    errors.lugarProcedencia = 'El lugar de procedencia es requerido'
  } else if (formData.value.lugarProcedencia.trim().length < 3) {
    errors.lugarProcedencia = 'Debe tener al menos 3 caracteres'
  }
  
  if (!formData.value.iglesia.trim()) {
    errors.iglesia = 'El nombre de la iglesia es requerido'
  } else if (formData.value.iglesia.trim().length < 5) {
    errors.iglesia = 'El nombre debe tener al menos 5 caracteres'
  }
  
  if (!formData.value.edad) {
    errors.edad = 'La edad es requerida'
  } else if (formData.value.edad < 18 || formData.value.edad > 100) {
    errors.edad = 'La edad debe estar entre 18 y 100 años'
  }

  if (tipoUsuario.value === 'pastor') {
    if (!formData.value.funcion) {
      errors.funcion = 'Debe seleccionar su función'
    }
  }
  
  if (tipoUsuario.value === 'miembro') {
    if (!formData.value.nombrePastor.trim()) {
      errors.nombrePastor = 'El nombre del pastor es requerido'
    } else if (formData.value.nombrePastor.trim().length < 5) {
      errors.nombrePastor = 'El nombre debe tener al menos 5 caracteres'
    }
    if (!formData.value.sexo) {
      errors.sexo = 'Debe seleccionar el sexo'
    }
  }

  const validateDate = (day, month, year, fieldName, minDate, maxDate) => {
    if (!day || !month || !year) {
      errors[fieldName] = 'Fecha completa requerida'
      return null
    }
    const date = new Date(`${year}-${month}-${day}T00:00:00`)
    if (isNaN(date.getTime())) {
      errors[fieldName] = 'Fecha inválida'
      return null
    }
    if (date < minDate || date > maxDate) {
      errors[fieldName] = `La fecha debe estar entre el ${minDate.getDate()} y ${maxDate.getDate()} de septiembre`
      return null
    }
    return date.toISOString().split('T')[0]
  }

  const minLlegada = new Date('2025-09-03T00:00:00')
  const maxLlegada = new Date('2025-09-04T23:59:59')
  formData.value.fechaLlegada = validateDate(formData.value.fechaLlegadaDay, formData.value.fechaLlegadaMonth, formData.value.fechaLlegadaYear, 'fechaLlegada', minLlegada, maxLlegada)

  const minRegreso = new Date('2025-09-07T00:00:00')
  const maxRegreso = new Date('2025-09-10T23:59:59')
  formData.value.fechaRegreso = validateDate(formData.value.fechaRegresoDay, formData.value.fechaRegresoMonth, formData.value.fechaRegresoYear, 'fechaRegreso', minRegreso, maxRegreso)

  if (formData.value.fechaLlegada && formData.value.fechaRegreso) {
    const llegada = new Date(formData.value.fechaLlegada)
    const regreso = new Date(formData.value.fechaRegreso)
    if (regreso <= llegada) {
      errors.fechaRegreso = 'La fecha de regreso debe ser posterior a la llegada'
    }
  }
  
  formErrors.value = errors
  return Object.keys(errors).length === 0
}

const getInputClass = (field) => {
  return [
    'w-full px-4 py-4 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
    formErrors.value[field] ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
  ].join(' ')
}

const getSelectClass = (field) => {
  return [
    'w-full px-4 py-4 rounded-lg bg-card text-foreground focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
    formErrors.value[field] ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
  ].join(' ')
}

const toggleEsposa = () => {
  showEsposaFields.value = !showEsposaFields.value
  if (!showEsposaFields.value) {
    formData.value.nombreEsposa = ''
    formData.value.edadEsposa = ''
  }
}

const toggleMemberSpouse = () => {
  showMemberSpouseFields.value = !showMemberSpouseFields.value
  if (!showMemberSpouseFields.value) {
    formData.value.nombreEsposoEsposa = ''
    formData.value.edadEsposoEsposa = ''
  }
}

const incrementAcompanantes = () => {
  const max = tipoUsuario.value === 'pastor' ? 15 : 10
  if (formData.value.totalAcompanantes < max) {
    formData.value.totalAcompanantes = parseInt(formData.value.totalAcompanantes) + 1
  }
}

const decrementAcompanantes = () => {
  if (formData.value.totalAcompanantes > 0) {
    formData.value.totalAcompanantes = parseInt(formData.value.totalAcompanantes) - 1
  }
}

const submitForm = async () => {
  if (!validateForm()) {
    submitMessage.value = 'Por favor, corrija los errores en el formulario'
    setTimeout(() => {
      submitMessage.value = ''
    }, 5000)
    return
  }
  
  isSubmitting.value = true
  submitMessage.value = ''
  
  try {
    const form = new FormData()
    
    form.append('tipoUsuario', tipoUsuario.value)
    form.append('nombre', formData.value.nombre.trim())
    form.append('apellido', formData.value.apellido.trim())
    form.append('telefono', formData.value.telefono.trim())
    form.append('lugarProcedencia', formData.value.lugarProcedencia.trim())
    form.append('edad', formData.value.edad)
    
    if (tipoUsuario.value === 'pastor') {
      form.append('funcion', formData.value.funcion)
      form.append('nombreEsposa', formData.value.nombreEsposa.trim() || '')
      form.append('edadEsposa', formData.value.edadEsposa || '')
    } else {
      form.append('nombrePastor', formData.value.nombrePastor.trim())
      form.append('sexo', formData.value.sexo)
      form.append('nombreEsposoEsposa', formData.value.nombreEsposoEsposa.trim() || '')
      form.append('edadEsposoEsposa', formData.value.edadEsposoEsposa || '')
      form.append('necesidadHospedaje', formData.value.necesidadHospedaje ? 'Sí (opcional)' : 'No')
    }
    
    form.append('iglesia', formData.value.iglesia.trim())
    form.append('fechaLlegada', formData.value.fechaLlegada)
    form.append('fechaRegreso', formData.value.fechaRegreso)
    form.append('agenciaViaje', formData.value.agenciaViaje.trim() || '')
    form.append('totalAcompanantes', totalAcompanantesCalculado.value.toString())
    form.append('tipoAcompanantes', formData.value.tipoAcompanantes || '')
    form.append('requerimientosEspeciales', formData.value.requerimientosEspeciales.trim() || '')
    form.append('fechaRegistro', new Date().toLocaleString('es-PE'))
    
    await fetch('https://script.google.com/macros/s/AKfycbwOtXcY0IBLHQJWA9S-3xGeQwwJemSW1OIKlZ6sWtktfToQRU1Q_WnfYOA-TShW7vEh/exec', {
      method: 'POST',
      mode: 'no-cors',
      body: form
    })
    
    submitMessage.value = '¡Registro exitoso! Nos pondremos en contacto contigo pronto para confirmar los detalles.'
    
    tipoUsuario.value = ''
    formData.value = {
      nombre: '',
      apellido: '',
      telefono: '',
      lugarProcedencia: '',
      funcion: '',
      edad: '',
      sexo: '',
      iglesia: '',
      nombrePastor: '',
      fechaLlegadaDay: '',
      fechaLlegadaMonth: '',
      fechaLlegadaYear: '',
      fechaRegresoDay: '',
      fechaRegresoMonth: '',
      fechaRegresoYear: '',
      agenciaViaje: '',
      nombreEsposa: '',
      edadEsposa: '',
      nombreEsposoEsposa: '',
      edadEsposoEsposa: '',
      totalAcompanantes: 0,
      tipoAcompanantes: '',
      necesidadHospedaje: false,
      requerimientosEspeciales: ''
    }
    formErrors.value = {}
    showEsposaFields.value = false
    showMemberSpouseFields.value = false
    
    document.getElementById('registro').scrollIntoView({ behavior: 'smooth' })
    
  } catch (error) {
    submitMessage.value = 'Error al enviar el registro. Por favor, verifique su conexión e intente nuevamente.'
    console.error('Error:', error)
  } finally {
    isSubmitting.value = false
    setTimeout(() => {
      submitMessage.value = ''
    }, 10000)
  }
}

const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value
}

const scrollToSection = (sectionId) => {
  const element = document.getElementById(sectionId)
  if (element) {
    element.scrollIntoView({ behavior: 'smooth' })
  }
  mobileMenuOpen.value = false
}

const toggleScheduleExpand = (time) => {
  expandedSchedule.value = expandedSchedule.value === time ? null : time
}

watch(() => formData.value.sexo, (newSexo) => {
  if (tipoUsuario.value === 'miembro') {
    showMemberSpouseFields.value = false
    formData.value.nombreEsposoEsposa = ''
    formData.value.edadEsposoEsposa = ''
  }
})

onMounted(() => {
  updateCountdown()
  countdownInterval = setInterval(updateCountdown, 1000)
})

onUnmounted(() => {
  if (countdownInterval) {
    clearInterval(countdownInterval)
  }
})
</script>


<template>
  <div class="min-h-screen bg-background">
    <nav class="bg-card/95 backdrop-blur-md shadow-lg border-b border-border sticky top-0 z-50">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-20">
          <div class="flex-shrink-0">
            <img :src="logo2" alt="Logo Primera Iglesia Bautista" class="h-16 w-auto" />
          </div>
          
          <div class="hidden md:block">
            <div class="ml-10 flex items-baseline space-x-8">
              <button
                @click="scrollToSection('inicio')"
                class="text-foreground-muted hover:text-foreground cursor-pointer px-4 py-2 text-sm font-semibold transition-all duration-300 relative group"
              >
                Inicio
                <span class="absolute inset-x-0 bottom-0 h-0.5 bg-primary transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300"></span>
              </button>
              <button
                @click="scrollToSection('programa')"
                class="text-foreground-muted hover:text-foreground cursor-pointer px-4 py-2 text-sm font-semibold transition-all duration-300 relative group"
              >
                Programa
                <span class="absolute inset-x-0 bottom-0 h-0.5 bg-primary transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300"></span>
              </button>
              <button
                @click="scrollToSection('hospedaje')"
                class="text-foreground-muted hover:text-foreground cursor-pointer px-4 py-2 text-sm font-semibold transition-all duration-300 relative group"
              >
                Hospedaje
                <span class="absolute inset-x-0 bottom-0 h-0.5 bg-primary transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300"></span>
              </button>
              <button
                @click="scrollToSection('registro')"
                class="bg-primary text-primary-foreground px-6 py-3 rounded-lg font-semibold hover:bg-primary-hover shadow-md hover:shadow-lg transform hover:-translate-y-0.5 transition-all duration-300"
              >
                Registro
              </button>
            </div>
          </div>
          
          <div class="md:hidden">
            <button @click="toggleMobileMenu" class="text-foreground-muted hover:text-foreground p-2 rounded-md hover:bg-card transition-colors">
              <Menu v-if="!mobileMenuOpen" class="h-6 w-6" />
              <X v-else class="h-6 w-6" />
            </button>
          </div>
        </div>
      </div>
      
      <div v-if="mobileMenuOpen" class="md:hidden bg-card/95 backdrop-blur-md border-t border-border shadow-lg">
        <div class="px-4 pt-4 pb-6 space-y-2">
          <button @click="scrollToSection('inicio')" class="block px-4 py-3 text-base font-semibold text-foreground-muted hover:text-foreground hover:bg-card rounded-md cursor-pointer transition-colors w-full text-left">Inicio</button>
          <button @click="scrollToSection('programa')" class="block px-4 py-3 text-base font-semibold text-foreground-muted hover:text-foreground hover:bg-card rounded-md cursor-pointer transition-colors w-full text-left">Programa</button>
          <button @click="scrollToSection('hospedaje')" class="block px-4 py-3 text-base font-semibold text-foreground-muted hover:text-foreground hover:bg-card rounded-md cursor-pointer transition-colors w-full text-left">Hospedaje</button>
          <button @click="scrollToSection('registro')" class="block mx-4 mt-4 bg-primary text-primary-foreground px-6 py-3 rounded-lg font-semibold text-center">Registro</button>
        </div>
      </div>
    </nav>

    <section id="inicio" class="relative min-h-screen flex items-center justify-center">
      <div class="absolute inset-0 z-0">
        <img :src="bgEvent" alt="Días de Predicación Misionera" class="w-full h-full object-cover" />
        <div class="absolute inset-0 bg-gradient-to-b from-primary/80 via-primary/70 to-primary/85"></div>
      </div>
      
      <div class="relative z-10 text-center text-primary-foreground px-4 max-w-5xl mx-auto">
        <div class="mb-12 max-w-4xl mx-auto">
          <h1 class="text-4xl md:text-5xl lg:text-6xl font-serif mb-8 leading-tight font-bold tracking-wide">
            <span class="text-primary-foreground">Días de</span>
            <span class="text-accent block md:inline"> Predicación </span>
            <span class="text-primary-foreground">Misionera</span>
          </h1>
          
          <div class="bg-primary-foreground/10 backdrop-blur-sm rounded-xl p-8 mb-8 border border-primary-foreground/20 shadow-2xl">
            <Quote class="h-8 w-8 mx-auto mb-4 opacity-60" />
            <p class="text-2xl md:text-3xl lg:text-4xl mb-4 italic font-light leading-relaxed">
              <span class="text-primary-foreground">"Las iglesias eran</span>
              <span class="text-accent font-medium"> confirmadas </span>
              <span class="text-primary-foreground">en la fe"</span>
            </p>
            <p class="text-xl font-semibold text-accent">Hechos 16:5</p>
          </div>
        </div>
        
        <p class="text-lg mb-4 text-primary-foreground opacity-90 font-medium">4-7 de Septiembre, 2025</p>
        <p class="text-lg mb-8 text-primary-foreground opacity-90 font-medium">Primera Iglesia Bautista de Huancayo</p>
        
        <div class="flex flex-col sm:flex-row gap-4 justify-center mb-12">
          <button
            @click="scrollToSection('programa')"
            class="bg-primary-foreground text-primary px-8 py-4 rounded-lg font-semibold text-lg hover:bg-card transition-all duration-300 shadow-lg hover:shadow-xl transform hover:-translate-y-1"
          >
            Ver Programa
          </button>
          <button
            @click="scrollToSection('ubicacion')"
            class="bg-transparent text-primary-foreground border-2 border-primary-foreground px-8 py-4 rounded-lg font-semibold text-lg hover:bg-primary-foreground hover:text-primary shadow-lg hover:shadow-xl transform hover:-translate-y-1 transition-all duration-300"
          >
            ¿Cómo llegar?
          </button>
        </div>
        
        <div class="bg-primary-foreground/10 backdrop-blur-sm rounded-xl p-8 max-w-2xl mx-auto border border-primary-foreground/20 shadow-2xl">
          <h3 class="text-xl font-semibold mb-6 text-accent">Faltan para el evento:</h3>
          <div class="grid grid-cols-4 gap-4">
            <div v-for="(value, key) in timeLeft" :key="key" class="text-center">
              <div class="text-3xl md:text-5xl font-bold text-primary-foreground mb-2">{{ value }}</div>
              <div class="text-sm uppercase tracking-wide text-primary-foreground opacity-80 font-medium">{{ key }}</div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section class="py-20 bg-card">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold">
            ¡Un tiempo para fortalecer a las iglesias en la fe y en la misión!
          </h2>
          <p class="text-xl text-foreground-muted max-w-4xl mx-auto leading-relaxed">
            La Primera Iglesia Bautista de Huancayo se complace en invitarle a nuestros Días de Predicación Misionera. 
            Durante estos días queremos animar a pastores, misioneros y hermanos en Cristo a perseverar en la obra del Señor,
            mientras nos gozamos en su Palabra y en la comunión fraternal.
          </p>
        </div>
        
        <div class="grid md:grid-cols-3 gap-8">
          <div class="text-center p-8 bg-background rounded-xl shadow-lg border border-border hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <Calendar class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground">Fechas</h3>
            <p class="text-foreground-muted text-lg">Del jueves 4 al domingo 7</p>
            <p class="text-sm text-foreground-muted mt-2">Septiembre 2025</p>
          </div>
          
          <div class="text-center p-8 bg-background rounded-xl shadow-lg border border-border hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <Mic class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground">Predicador Principal</h3>
            <p class="text-foreground-muted text-lg font-semibold">Hno. Daniel Rose</p>
            <p class="text-sm text-foreground-muted mt-2">Ministerio Internacional</p>
          </div>
          
          <div class="text-center p-8 bg-background rounded-xl shadow-lg border border-border hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <MapPin class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground">Ubicación</h3>
            <p class="text-foreground-muted text-lg">Av. Leandra Torres 263</p>
            <p class="text-sm text-foreground-muted mt-2">Huancayo, Perú</p>
          </div>
        </div>
      </div>
    </section>

    <section id="programa" class="py-20 bg-background">
      <div class="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold">
            Programa General
          </h2>
          <p class="text-xl text-foreground-muted">Predicador principal: <strong class="text-foreground">Hno. Daniel Rose</strong></p>
        </div>
        
        <div class="bg-card rounded-2xl p-8 shadow-xl border border-border mb-12">
          <h3 class="text-2xl font-bold text-foreground mb-6 text-center">Horarios de Servicios</h3>
          <div class="grid md:grid-cols-2 gap-6">
            <div class="space-y-4">
              <div class="flex justify-between items-center p-4 bg-background rounded-lg">
                <span class="font-semibold text-foreground">Jueves y Viernes:</span>
                <span class="text-foreground-muted">7:00 p.m.</span>
              </div>
              <div class="flex justify-between items-center p-4 bg-background rounded-lg">
                <span class="font-semibold text-foreground">Sábado:</span>
                <span class="text-foreground-muted">6:00 p.m.</span>
              </div>
            </div>
            <div class="space-y-4">
              <div class="flex justify-between  items-center p-4 bg-badge rounded-lg border border-accent bg-accent/10 border-badge">
                <span class="font-semibold text-foreground">Ciclo Misionero:</span>
                <span class="text-foreground-muted">Vie y Sáb 10:00 a.m.</span>
              </div>
              <div class="flex justify-between items-center p-4 bg-background rounded-lg">
                <span class="font-semibold  text-white">Domingo:</span>
                <span class="text-foreground-muted">10:00 a.m. y 6:00 p.m.</span>
              </div>
            </div>
          </div>
        </div>
        
        <div class="flex flex-wrap justify-center mb-12 bg-card rounded-xl p-2 max-w-2xl mx-auto shadow-inner">
          <button
            v-for="(label, day) in dayLabels"
            :key="day"
            @click="activeDay = day"
            :class="[
              'px-6 py-3 font-semibold text-sm transition-all duration-300 rounded-lg mx-1 my-1',
              activeDay === day
                ? 'bg-primary text-primary-foreground shadow-lg transform scale-105'
                : 'text-foreground-muted hover:text-foreground hover:bg-card hover:shadow-md'
            ]"
          >
            {{ label }}
          </button>
        </div>
        
        <div class="space-y-4">
          <div
            v-for="item in scheduleData[activeDay]"
            :key="item.time"
            class="bg-card rounded-xl border-2 border-border overflow-hidden hover:border-accent transition-all duration-300 shadow-md hover:shadow-lg"
          >
            <div
              @click="toggleScheduleExpand(item.time)"
              class="flex items-center justify-between p-8 cursor-pointer hover:bg-background transition-colors duration-200"
            >
              <div class="flex items-center space-x-6">
                <div class="flex items-center text-foreground bg-background rounded-lg px-4 py-2 shadow-sm border border-border">
                  <Clock class="h-5 w-5 mr-3 text-foreground-muted" />
                  <span class="font-bold text-xl">{{ item.time }}</span>
                </div>
                <div>
                  <h3 class="font-bold text-xl text-foreground mb-1">{{ item.activity }}</h3>
                </div>
              </div>
              <ChevronDown
                :class="[
                  'h-6 w-6 text-foreground-muted transition-transform duration-300',
                  expandedSchedule === item.time ? 'rotate-180' : ''
                ]"
              />
            </div>
            
            <div
              v-if="expandedSchedule === item.time"
              class="px-8 pb-8 text-foreground-muted border-t border-border bg-background/50"
            >
              <p class="mt-6 text-lg leading-relaxed">{{ item.description }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="hospedaje" class="py-20 bg-card">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold">
            Hospedaje y Alimentación
          </h2>
          <p class="text-xl text-foreground-muted max-w-3xl mx-auto leading-relaxed">
            Ofrecemos alojamiento y alimentación para hacer de su estancia una experiencia cómoda y bendecida
          </p>
        </div>
        
        <div class="grid lg:grid-cols-2 gap-8 mb-12">
          <div class="bg-background rounded-2xl shadow-xl border border-border p-8 hover:shadow-2xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <Church class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground text-center">Pastores y Misioneros</h3>
            <ul class="space-y-3 text-foreground-muted">
              <li class="flex items-start space-x-2">
                <CheckCircle class="h-5 w-5 text-green-600 mt-0.5 flex-shrink-0" />
                <span>Alojamiento garantizado y gratuito</span>
              </li>
              <li class="flex items-start space-x-2">
                <CheckCircle class="h-5 w-5 text-green-600 mt-0.5 flex-shrink-0" />
                <span>Para usted y su esposa</span>
              </li>
              <li class="flex items-start space-x-2">
                <CheckCircle class="h-5 w-5 text-green-600 mt-0.5 flex-shrink-0" />
                <span>Alimentación completa incluida</span>
              </li>
              <li class="flex items-start space-x-2">
                <CheckCircle class="h-5 w-5 text-green-600 mt-0.5 flex-shrink-0" />
                <span>Habitaciones cómodas y privadas</span>
              </li>
            </ul>
          </div>
          
          <div class="bg-background rounded-2xl shadow-xl border border-border p-8 hover:shadow-2xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <Users class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground text-center">Hermanos Visitantes</h3>
            <ul class="space-y-3 text-foreground-muted">
              <li class="flex items-start space-x-2">
                <Heart class="h-5 w-5 text-red-500 mt-0.5 flex-shrink-0" />
                <span>Hospedaje solidario opcional</span>
              </li>
              <li class="flex items-start space-x-2">
                <Heart class="h-5 w-5 text-red-500 mt-0.5 flex-shrink-0" />
                <span>Familias anfitrionas disponibles</span>
              </li>
              <li class="flex items-start space-x-2">
                <Heart class="h-5 w-5 text-red-500 mt-0.5 flex-shrink-0" />
                <span>Refrigerios durante eventos</span>
              </li>
              <li class="flex items-start space-x-2">
                <Heart class="h-5 w-5 text-red-500 mt-0.5 flex-shrink-0" />
                <span>Comunión fraternal</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="bg-gradient-to-r from-primary to-primary-hover rounded-2xl p-8 text-primary-foreground shadow-2xl">
          <div class="text-center mb-8">
            <Building class="h-12 w-12 mx-auto mb-4 text-accent" />
            <h3 class="text-3xl font-bold mb-4">Hotel Asignado</h3>
            <p class="text-xl text-primary-foreground/90">Ubicación del hospedaje que les proporcionaremos</p>
          </div>
          
          <div class="grid md:grid-cols-2 gap-8 items-center">
            <div>
              <h4 class="text-2xl font-bold mb-4 text-accent">Hotel Presidente Huancayo</h4>
              <div class="space-y-3 mb-6">
                <div class="flex items-center space-x-3">
                  <MapPin class="h-5 w-5 text-accent" />
                  <span>Real 1138, Huancayo (a 5 min de la iglesia)</span>
                </div>
                <div class="flex items-center space-x-3">
                  <Star class="h-5 w-5 text-accent" />
                  <span>Hotel 4 estrellas</span>
                </div>
                <div class="flex items-center space-x-3">
                  <Phone class="h-5 w-5 text-accent" />
                  <span>+51 64 231 275</span>
                </div>
              </div>
              
              <div class="grid grid-cols-2 gap-4 text-sm">
                <div class="flex items-center space-x-2">
                  <Wifi class="h-4 w-4 text-green-400" />
                  <span>WiFi gratuito</span>
                </div>
                <div class="flex items-center space-x-2">
                  <Car class="h-4 w-4 text-green-400" />
                  <span>Estacionamiento</span>
                </div>
                <div class="flex items-center space-x-2">
                  <Coffee class="h-4 w-4 text-green-400" />
                  <span>Desayuno incluido</span>
                </div>
                <div class="flex items-center space-x-2">
                  <Shield class="h-4 w-4 text-green-400" />
                  <span>Seguridad 24h</span>
                </div>
              </div>
            </div>
            
            <div class="bg-primary-foreground/10 rounded-xl p-6 backdrop-blur-sm">
              <h5 class="text-lg font-bold mb-4 text-accent">Información Importante</h5>
              <div class="space-y-3 text-sm">
                <p>• Este es el hotel donde les proporcionaremos hospedaje</p>
                <p>• No necesita hacer reservas por su cuenta</p>
                <p>• Nosotros nos encargamos de todos los arreglos</p>
                <p>• Solo para su referencia de ubicación</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="predicadores" class="py-20 bg-background">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold">
            Pastores Participantes
          </h2>
          <p class="text-xl text-foreground-muted max-w-3xl mx-auto leading-relaxed">
            Siervos de Dios comprometidos con la predicación fiel de Su Palabra
          </p>
        </div>
        
        <div class="grid lg:grid-cols-3 gap-10">
          <div v-for="pastor in pastores" :key="pastor.nombre"
                class="bg-card rounded-2xl shadow-xl border border-border hover:shadow-2xl transition-all duration-500 transform hover:-translate-y-3 overflow-hidden group">
            
            <div class="relative overflow-hidden">
              <img :src="pastor.imagen" :alt="pastor.nombre"
                    class="w-full h-80 object-cover transition-transform duration-500 group-hover:scale-105" />
              <div class="absolute inset-0 bg-primary/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
            </div>
            
            <div class="p-8">
              <div class="text-center mb-6">
                <h3 class="text-2xl font-bold text-foreground mb-2">{{ pastor.nombre }}</h3>
                <p class="text-foreground-muted font-semibold text-lg mb-1">{{ pastor.rol }}</p>
                <p class="text-foreground-muted text-sm">{{ pastor.iglesia }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="ubicacion" class="py-20 bg-card">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold">
            Ubicación
          </h2>
          <p class="text-xl text-foreground-muted">Te esperamos en nuestra iglesia</p>
        </div>
        
        <div class="grid lg:grid-cols-3 gap-12">
          <div class="lg:col-span-1">
            <div class="bg-background p-8 rounded-2xl shadow-xl border border-border h-full">
              <h3 class="text-2xl font-bold mb-8 text-foreground">Información de Contacto</h3>
              
              <div class="space-y-6">
                <div class="flex items-start space-x-4 p-4 bg-card rounded-lg shadow-sm">
                  <div class="bg-primary rounded-full w-12 h-12 flex items-center justify-center flex-shrink-0">
                    <MapPin class="h-6 w-6 text-primary-foreground" />
                  </div>
                  <div>
                    <p class="font-bold text-foreground mb-1">Dirección</p>
                    <p class="text-foreground-muted">Av. Leandra Torres 263<br>Huancayo, Perú</p>
                  </div>
                </div>
                
                <div class="flex items-start space-x-4 p-4 bg-card rounded-lg shadow-sm">
                  <div class="bg-primary rounded-full w-12 h-12 flex items-center justify-center flex-shrink-0">
                    <MessageCircle class="h-6 w-6 text-primary-foreground" />
                  </div>
                  <div>
                    <p class="font-bold text-foreground mb-1">WhatsApp</p>
                    <p class="text-foreground-muted">+51 999 999 999</p>
                  </div>
                </div>
                
                <div class="flex items-start space-x-4 p-4 bg-card rounded-lg shadow-sm">
                  <div class="bg-primary rounded-full w-12 h-12 flex items-center justify-center flex-shrink-0">
                    <Globe class="h-6 w-6 text-primary-foreground" />
                  </div>
                  <div>
                    <p class="font-bold text-foreground mb-1">Web</p>
                    <p class="text-foreground-muted">www.pibhyo.org</p>
                  </div>
                </div>
              </div>
              
              <a class="block w-full mt-8 bg-primary text-primary-foreground px-6 py-4 rounded-lg font-semibold text-center hover:bg-primary-hover shadow-lg hover:shadow-xl transform hover:-translate-y-1 transition-all duration-300" target="_blank" href="https://maps.app.goo.gl/43H5Q16MrWckdVks6">
                Ver en Google Maps
              </a>
            </div>
          </div>
          
          <div class="lg:col-span-2">
            <div class="rounded-2xl overflow-hidden shadow-xl border-2 border-border h-96 lg:h-full">
              <iframe
                src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d243.86087300215016!2d-75.2035786184613!3d-12.059067140759481!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x910e96357221f9c9%3A0x8f3ef533256ff91e!2sAv.%20Leandra%20Torres%20263%2C%20Huancayo%2012001!5e0!3m2!1ses-419!2spe!4v1754681689797!5m2!1ses-419!2spe"
                width="100%"
                height="100%"
                style="border:0;"
                allowfullscreen=""
                loading="lazy"
                referrerpolicy="no-referrer-when-downgrade"
                class="w-full h-full rounded-2xl">
              </iframe>
            </div>
          </div>
        </div>
      </div>
    </section>

    <section id="registro" class="py-20 bg-primary text-primary-foreground">
      <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-12">
          <h2 class="text-4xl md:text-5xl font-serif mb-6 font-bold">
            Inscripción Gratuita
          </h2>
          <p class="text-xl mb-4 opacity-90">
            Complete el formulario según su función ministerial
          </p>
          <p class="text-lg opacity-75">
            4-7 de Septiembre • Primera Iglesia Bautista de Huancayo
          </p>
        </div>
        
        <div class="max-w-4xl mx-auto bg-primary-foreground/10 backdrop-blur-sm rounded-2xl p-6 md:p-10 border border-primary-foreground/20 shadow-2xl">
          <div class="mb-8 pb-8 border-b border-primary-foreground/20">
            <h3 class="text-2xl font-bold mb-6 text-center text-accent">Tipo de Registro</h3>
            <div class="grid md:grid-cols-2 gap-4">
              <button
                @click="tipoUsuario = 'pastor'"
                :class="[
                  'p-6 rounded-xl border-2 transition-all duration-300 text-left group',
                  tipoUsuario === 'pastor'
                    ? 'border-primary-foreground bg-primary-foreground/20'
                    : 'border-primary-foreground/30 hover:border-primary-foreground/60'
                ]"
              >
                <div class="flex items-center mb-3">
                  <Church class="h-6 w-6 mr-3 text-accent" />
                  <h4 class="font-bold text-lg text-accent">Pastor o Misionero</h4>
                </div>
                <p class="text-sm opacity-80">Para pastores, misioneros y líderes ministeriales</p>
                <div class="mt-3 text-xs opacity-70">
                  • Hospedaje garantizado incluido<br>
                  • Campos adicionales para esposa
                </div>
              </button>
              
              <button
                @click="tipoUsuario = 'miembro'"
                :class="[
                  'p-6 rounded-xl border-2 transition-all duration-300 text-left group',
                  tipoUsuario === 'miembro'
                    ? 'border-primary-foreground bg-primary-foreground/20'
                    : 'border-primary-foreground/30 hover:border-primary-foreground/60'
                ]"
              >
                <div class="flex items-center mb-3">
                  <Users class="h-6 w-6 mr-3 text-accent" />
                  <h4 class="font-bold text-lg text-accent">Miembro o Invitado</h4>
                </div>
                <p class="text-sm opacity-80">Para miembros de iglesia e invitados</p>
                <div class="mt-3 text-xs opacity-70">
                  • Hospedaje solidario opcional<br>
                  • Formulario simplificado
                </div>
              </button>
            </div>
            <p v-if="formErrors.tipoUsuario" class="text-red-400 text-sm mt-2 text-center">{{ formErrors.tipoUsuario }}</p>
          </div>

          <div v-if="tipoUsuario">
            <form @submit.prevent="submitForm" class="space-y-8">
              <div class="border-b border-primary-foreground/20 pb-8">
                <h3 class="text-2xl font-bold mb-6 text-accent">Información Personal</h3>
                <div class="grid md:grid-cols-2 gap-6">
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Nombres *</label>
                    <input
                      type="text"
                      v-model="formData.nombre"
                      placeholder="Sus nombres completos"
                      :class="getInputClass('nombre')"
                    >
                    <p v-if="formErrors.nombre" class="text-red-400 text-sm mt-2">{{ formErrors.nombre }}</p>
                  </div>
                  
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Apellidos *</label>
                    <input
                      type="text"
                      v-model="formData.apellido"
                      placeholder="Sus apellidos completos"
                      :class="getInputClass('apellido')"
                    >
                    <p v-if="formErrors.apellido" class="text-red-400 text-sm mt-2">{{ formErrors.apellido }}</p>
                  </div>
                </div>
                
                <div class="grid md:grid-cols-2 gap-6 mt-6">
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Número de celular *</label>
                    <input
                      type="tel"
                      v-model="formData.telefono"
                      placeholder="+51 999 999 999"
                      :class="getInputClass('telefono')"
                    >
                    <p v-if="formErrors.telefono" class="text-red-400 text-sm mt-2">{{ formErrors.telefono }}</p>
                  </div>
                  
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Lugar de procedencia *</label>
                    <input
                      type="text"
                      v-model="formData.lugarProcedencia"
                      placeholder="Ciudad, región"
                      :class="getInputClass('lugarProcedencia')"
                    >
                    <p v-if="formErrors.lugarProcedencia" class="text-red-400 text-sm mt-2">{{ formErrors.lugarProcedencia }}</p>
                  </div>
                </div>

                <div v-if="tipoUsuario === 'pastor'" class="grid md:grid-cols-2 gap-6 mt-6">
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Función *</label>
                    <select v-model="formData.funcion" :class="getSelectClass('funcion')">
                      <option value="">Seleccione</option>
                      <option value="pastor">Pastor</option>
                      <option value="misionero">Misionero</option>
                    </select>
                    <p v-if="formErrors.funcion" class="text-red-400 text-sm mt-2">{{ formErrors.funcion }}</p>
                  </div>
                  
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Edad *</label>
                    <input
                      type="number"
                      v-model="formData.edad"
                      placeholder="Edad"
                      min="18" max="100"
                      :class="getInputClass('edad')"
                    >
                    <p v-if="formErrors.edad" class="text-red-400 text-sm mt-2">{{ formErrors.edad }}</p>
                  </div>
                </div>

                <div v-if="tipoUsuario === 'miembro'" class="grid md:grid-cols-2 gap-6 mt-6">
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Edad *</label>
                    <input
                      type="number"
                      v-model="formData.edad"
                      placeholder="Edad"
                      min="18" max="100"
                      :class="getInputClass('edad')"
                    >
                    <p v-if="formErrors.edad" class="text-red-400 text-sm mt-2">{{ formErrors.edad }}</p>
                  </div>
                  
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Sexo *</label>
                    <select v-model="formData.sexo" :class="getSelectClass('sexo')">
                      <option value="">Seleccione</option>
                      <option value="masculino">Masculino</option>
                      <option value="femenino">Femenino</option>
                    </select>
                    <p v-if="formErrors.sexo" class="text-red-400 text-sm mt-2">{{ formErrors.sexo }}</p>
                  </div>
                </div>
              </div>

              <div class="border-b border-primary-foreground/20 pb-8">
                <h3 class="text-2xl font-bold mb-6 text-accent">Información Ministerial</h3>
                <div class="space-y-6">
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Nombre de su iglesia *</label>
                    <input
                      type="text"
                      v-model="formData.iglesia"
                      placeholder="Nombre completo de la iglesia"
                      :class="getInputClass('iglesia')"
                    >
                    <p v-if="formErrors.iglesia" class="text-red-400 text-sm mt-2">{{ formErrors.iglesia }}</p>
                  </div>
                  
                  <div v-if="tipoUsuario === 'miembro'">
                    <label class="block text-sm font-semibold mb-3 opacity-90">Nombre de su pastor *</label>
                    <input
                      type="text"
                      v-model="formData.nombrePastor"
                      placeholder="Nombre completo del pastor"
                      :class="getInputClass('nombrePastor')"
                    >
                    <p v-if="formErrors.nombrePastor" class="text-red-400 text-sm mt-2">{{ formErrors.nombrePastor }}</p>
                  </div>
                </div>
              </div>

              <div class="border-b border-primary-foreground/20 pb-8">
                <h3 class="text-2xl font-bold mb-6 text-accent">Información de Viaje</h3>
                <div class="grid md:grid-cols-2 gap-6">
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Fecha de llegada *</label>
                    <div class="grid grid-cols-3 gap-2">
                      <select v-model="formData.fechaLlegadaDay" :class="getSelectClass('fechaLlegada')">
                        <option value="">Día</option>
                        <option v-for="day in 31" :key="day" :value="String(day).padStart(2, '0')">{{ day }}</option>
                      </select>
                      <select v-model="formData.fechaLlegadaMonth" :class="getSelectClass('fechaLlegada')">
                        <option value="">Mes</option>
                        <option v-for="(month, index) in months" :key="index" :value="String(index + 1).padStart(2, '0')">{{ month }}</option>
                      </select>
                      <select v-model="formData.fechaLlegadaYear" :class="getSelectClass('fechaLlegada')">
                        <option value="">Año</option>
                        <option value="2025">2025</option>
                      </select>
                    </div>
                    <p v-if="formErrors.fechaLlegada" class="text-red-400 text-sm mt-2">{{ formErrors.fechaLlegada }}</p>
                    <p class="text-xs opacity-70 mt-1">Entre el 3 y 4 de septiembre</p>
                  </div>
                  
                  <div>
                    <label class="block text-sm font-semibold mb-3 opacity-90">Fecha de regreso *</label>
                    <div class="grid grid-cols-3 gap-2">
                      <select v-model="formData.fechaRegresoDay" :class="getSelectClass('fechaRegreso')">
                        <option value="">Día</option>
                        <option v-for="day in 31" :key="day" :value="String(day).padStart(2, '0')">{{ day }}</option>
                      </select>
                      <select v-model="formData.fechaRegresoMonth" :class="getSelectClass('fechaRegreso')">
                        <option value="">Mes</option>
                        <option v-for="(month, index) in months" :key="index" :value="String(index + 1).padStart(2, '0')">{{ month }}</option>
                      </select>
                      <select v-model="formData.fechaRegresoYear" :class="getSelectClass('fechaRegreso')">
                        <option value="">Año</option>
                        <option value="2025">2025</option>
                      </select>
                    </div>
                    <p v-if="formErrors.fechaRegreso" class="text-red-400 text-sm mt-2">{{ formErrors.fechaRegreso }}</p>
                    <p class="text-xs opacity-70 mt-1">Entre el 7 y 10 de septiembre</p>
                  </div>
                </div>
                
                <div class="mt-6">
                  <label class="block text-sm font-semibold mb-3 opacity-90">Agencia de viaje (opcional)</label>
                  <input
                    type="text"
                    v-model="formData.agenciaViaje"
                    placeholder="Nombre de la agencia de viajes"
                    class="w-full px-4 py-4 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2 border-transparent focus:border-primary-foreground"
                  >
                </div>
              </div>

              <div class="border-b border-primary-foreground/20 pb-8">
                <div class="flex items-center justify-between mb-6">
                  <h3 class="text-2xl font-bold text-accent">Acompañantes</h3>
                  <div class="flex items-center space-x-2 text-sm opacity-75">
                    <Users class="h-4 w-4" />
                    <span>Total: {{ totalAcompanantesCalculado }}</span>
                  </div>
                </div>
                
                <div v-if="tipoUsuario === 'pastor'" class="space-y-6">
                  <div class="bg-primary-foreground/5 rounded-lg p-6 border border-primary-foreground/20">
                    <div class="flex items-center justify-between mb-4">
                      <h4 class="font-bold text-lg text-accent">Esposa</h4>
                      <button
                        type="button"
                        @click="toggleEsposa"
                        :class="[
                          'px-4 py-2 rounded-lg text-sm font-semibold transition-all',
                          showEsposaFields ? 'bg-green-500/20 text-green-300' : 'bg-primary-foreground/20 text-primary-foreground'
                        ]"
                      >
                        {{ showEsposaFields ? 'Quitar Esposa' : 'Agregar Esposa' }}
                      </button>
                    </div>
                    
                    <div v-if="showEsposaFields" class="grid md:grid-cols-2 gap-4">
                      <input
                        type="text"
                        v-model="formData.nombreEsposa"
                        placeholder="Nombre completo de la esposa"
                        class="px-4 py-3 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground"
                      >
                      <input
                        type="number"
                        v-model="formData.edadEsposa"
                        placeholder="Edad"
                        min="18" max="100"
                        class="px-4 py-3 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground"
                      >
                    </div>
                  </div>

                  <div class="bg-primary-foreground/5 rounded-lg p-6 border border-primary-foreground/20">
                    <h4 class="font-bold text-lg mb-4 text-accent">Otros Acompañantes</h4>
                    
                    <div class="grid md:grid-cols-2 gap-6">
                      <div>
                        <label class="block text-sm font-semibold mb-3 opacity-90">Número adicional</label>
                        <div class="flex items-center space-x-3">
                          <button
                            type="button"
                            @click="decrementAcompanantes"
                            :disabled="formData.totalAcompanantes <= 0"
                            class="w-10 h-10 rounded-lg bg-card text-foreground font-bold hover:bg-background transition-all disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center"
                          >
                            <Minus class="h-4 w-4" />
                          </button>
                          <input
                            type="number"
                            v-model="formData.totalAcompanantes"
                            placeholder="0"
                            min="0" max="15"
                            class="flex-1 px-4 py-3 rounded-lg bg-card text-foreground text-center font-bold text-lg focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground"
                          >
                          <button
                            type="button"
                            @click="incrementAcompanantes"
                            :disabled="formData.totalAcompanantes >= 15"
                            class="w-10 h-10 rounded-lg bg-card text-foreground font-bold hover:bg-background transition-all disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center"
                          >
                            <Plus class="h-4 w-4" />
                          </button>
                        </div>
                        <p class="text-xs opacity-70 mt-1">Máximo 15 personas adicionales</p>
                      </div>
                      
                      <div>
                        <label class="block text-sm font-semibold mb-3 opacity-90">Composición del grupo</label>
                        <select v-model="formData.tipoAcompanantes" class="w-full px-4 py-3 rounded-lg bg-card text-foreground focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground">
                          <option value="">Seleccione</option>
                          <option value="solo-varones">Solo varones</option>
                          <option value="solo-damas">Solo damas</option>
                          <option value="matrimonios">Matrimonios</option>
                          <option value="adultos">Solo adultos</option>
                          <option value="niños">Solo niños</option>
                          <option value="adultos-niños">1 adulto y 1 niño</option>
                          <option value="mixto-general">Mixto (varios tipos)</option>
                        </select>
                      </div>
                    </div>
                  </div>
                </div>

                <div v-if="tipoUsuario === 'miembro'" class="space-y-6">
                  <div class="bg-primary-foreground/5 rounded-lg p-6 border border-primary-foreground/20">
                    <h4 class="font-bold text-lg mb-4 text-accent">Acompañantes</h4>
                    
                    <div class="grid md:grid-cols-2 gap-6">
                      <div>
                        <label class="block text-sm font-semibold mb-3 opacity-90">Número de personas</label>
                        <div class="flex items-center space-x-3">
                          <button
                            type="button"
                            @click="decrementAcompanantes"
                            :disabled="formData.totalAcompanantes <= 0"
                            class="w-10 h-10 rounded-lg bg-card text-foreground font-bold hover:bg-background transition-all disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center"
                          >
                            <Minus class="h-4 w-4" />
                          </button>
                          <input
                            type="number"
                            v-model="formData.totalAcompanantes"
                            placeholder="0"
                            min="0" max="10"
                            class="flex-1 px-4 py-3 rounded-lg bg-card text-foreground text-center font-bold text-lg focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground"
                          >
                          <button
                            type="button"
                            @click="incrementAcompanantes"
                            :disabled="formData.totalAcompanantes >= 10"
                            class="w-10 h-10 rounded-lg bg-card text-foreground font-bold hover:bg-background transition-all disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center"
                          >
                            <Plus class="h-4 w-4" />
                          </button>
                        </div>
                        <p class="text-xs opacity-70 mt-1">Máximo 10 acompañantes</p>
                      </div>
                      
                      <div>
                        <label class="block text-sm font-semibold mb-3 opacity-90">Tipo de grupo</label>
                        <select v-model="formData.tipoAcompanantes" class="w-full px-4 py-3 rounded-lg bg-card text-foreground focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground">
                          <option value="">Seleccione</option>
                          <option value="familiares">Familiares</option>
                          <option value="amigos">Amigos de la iglesia</option>
                          <option value="matrimonios">Matrimonios</option>
                          <option value="solo-varones">Solo varones</option>
                          <option value="solo-mujeres">Solo mujeres</option>
                          <option value="adultos">Solo adultos</option>
                          <option value="niños">Solo niños</option>
                          <option value="adultos-niños">1 adulto y 1 niño</option>
                          <option value="mixto-general">Mixto (varios tipos)</option>
                        </select>
                      </div>
                    </div>

                    <div v-if="formData.sexo" class="bg-primary-foreground/5 rounded-lg p-6 border border-primary-foreground/20 mt-6">
                      <div class="flex items-center justify-between mb-4">
                        <h4 class="font-bold text-lg text-accent">{{ spouseLabel }}</h4>
                        <button
                          type="button"
                          @click="toggleMemberSpouse"
                          :class="[
                            'px-4 py-2 rounded-lg text-sm font-semibold transition-all',
                            showMemberSpouseFields ? 'bg-green-500/20 text-green-300' : 'bg-primary-foreground/20 text-primary-foreground'
                          ]"
                        >
                          {{ showMemberSpouseFields ? `Quitar ${spouseLabel}` : `Agregar ${spouseLabel}` }}
                        </button>
                      </div>
                      
                      <div v-if="showMemberSpouseFields" class="grid md:grid-cols-2 gap-4">
                        <input
                          type="text"
                          v-model="formData.nombreEsposoEsposa"
                          :placeholder="`Nombre completo de ${spouseLabel.toLowerCase()}`"
                          class="px-4 py-3 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground"
                        >
                        <input
                          type="number"
                          v-model="formData.edadEsposoEsposa"
                          placeholder="Edad"
                          min="18" max="100"
                          class="px-4 py-3 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-2 focus:ring-primary-foreground/50 transition-all border-2 border-transparent focus:border-primary-foreground"
                        >
                      </div>
                    </div>

                    <div class="mt-6">
                      <label class="flex items-start space-x-3">
                        <input
                          type="checkbox"
                          v-model="formData.necesidadHospedaje"
                          class="w-5 h-5 text-primary bg-card border-2 border-primary-foreground/30 rounded focus:ring-primary-foreground/50 focus:ring-2 mt-1"
                        >
                        <div>
                          <span class="text-sm font-semibold opacity-90 flex items-center">
                            <Heart class="h-4 w-4 mr-2" />
                            Solicito hospedaje solidario (opcional)
                          </span>
                          <p class="text-xs opacity-70 mt-1">Sujeto a disponibilidad de familias anfitrionas. Nos contactaremos si es posible.</p>
                        </div>
                      </label>
                    </div>
                  </div>
                </div>
              </div>

              <div class="border-b border-primary-foreground/20 pb-8">
                <h3 class="text-2xl font-bold mb-6 text-accent">Requerimientos Especiales</h3>
                <div>
                  <label class="block text-sm font-semibold mb-3 opacity-90">¿Tiene algún requerimiento especial? (opcional)</label>
                  <textarea
                    v-model="formData.requerimientosEspeciales"
                    placeholder="Ej: Alergias alimentarias, necesidades médicas, accesibilidad, etc."
                    rows="4"
                    class="w-full px-4 py-4 rounded-lg bg-card text-foreground placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2 border-transparent focus:border-primary-foreground resize-none"
                  ></textarea>
                  <p class="text-xs opacity-70 mt-2">Indique cualquier necesidad especial de salud, accesibilidad, etc.</p>
                </div>
              </div>

              <div v-if="submitMessage" :class="[
                'text-center p-4 rounded-lg border',
                submitMessage.includes('exitoso')
                  ? 'bg-green-500/20 text-green-300 border-green-500/30'
                  : 'bg-red-500/20 text-red-300 border-red-500/30'
              ]">
                <div class="flex items-center justify-center space-x-2">
                  <CheckCircle v-if="submitMessage.includes('exitoso')" class="h-5 w-5" />
                  <AlertCircle v-else class="h-5 w-5" />
                  <span>{{ submitMessage }}</span>
                </div>
              </div>

              <button
                type="submit"
                :disabled="isSubmitting"
                :class="[
                  'w-full px-8 py-5 rounded-lg font-bold text-xl transition-all duration-300 shadow-lg hover:shadow-xl transform flex items-center justify-center space-x-2',
                  isSubmitting
                    ? 'bg-gray-400 text-gray-600 cursor-not-allowed'
                    : 'bg-primary-foreground text-primary hover:bg-card hover:-translate-y-1'
                ]"
              >
                <Loader v-if="isSubmitting" class="h-5 w-5 animate-spin" />
                <Send v-else class="h-5 w-5" />
                <span>{{ isSubmitting ? 'Enviando registro...' : 'Confirmar Registro Gratuito' }}</span>
              </button>
            </form>
          </div>

          <div v-if="!tipoUsuario" class="text-center py-12">
            <Users class="h-16 w-16 mx-auto mb-4 opacity-60" />
            <h3 class="text-xl font-bold mb-2 text-accent">Seleccione su tipo de registro</h3>
            <p class="text-lg opacity-75">Por favor, elija una opción arriba para continuar con el formulario</p>
          </div>
          
          <div class="mt-10 pt-8 border-t border-primary-foreground/20 text-center">
            <p class="opacity-75 mb-3">¿Tienes preguntas sobre el registro?</p>
            <div class="space-y-2">
              <p class="font-bold text-xl text-accent">📱 WhatsApp: +51 999 999 999</p>
              <p class="font-bold text-xl text-accent">🌐 Web: www.pibhyo.org</p>
            </div>
            <p class="text-sm opacity-60 mt-4">Horario de atención: Lunes a Viernes 9:00 AM - 6:00 PM</p>
          </div>
        </div>
      </div>
    </section>

    <section class="py-16 bg-primary-hover text-primary-foreground">
      <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
        <Quote class="h-12 w-12 mx-auto mb-6 opacity-60" />
        <h2 class="text-3xl md:text-4xl font-serif mb-6 font-bold">
          ¡Ora por este evento!
        </h2>
        <p class="text-xl mb-6 opacity-90 leading-relaxed">
          Pedimos tus oraciones para que estos días sean usados por el Señor para confirmar a su iglesia en la fe 
          y para enviar más obreros a la mies.
        </p>
        <p class="text-2xl font-bold text-accent">
          ¡Te esperamos en Huancayo con gozo en el Señor!
        </p>
      </div>
    </section>

    <footer class="bg-primary text-primary-foreground py-16 border-t-4 border-primary-hover">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="grid md:grid-cols-3 gap-12">
          <div>
            <img :src="logo" alt="Logo Primera Iglesia Bautista" class="h-16 w-auto mb-6 filter brightness-0 invert" />
            <p class="opacity-90 leading-relaxed text-lg">
              Proclamando fielmente la Palabra de Dios para la gloria de Cristo y la edificación de Su iglesia.
            </p>
            <p class="font-semibold mt-4 italic text-accent">"Soli Deo Gloria"</p>
          </div>
          
          <div>
            <h3 class="text-2xl font-bold mb-6 text-accent">Contacto</h3>
            <div class="space-y-4 opacity-90">
              <div class="flex items-center space-x-3">
                <MessageCircle class="h-5 w-5 text-accent" />
                <p>+51 999 999 999</p>
              </div>
              <div class="flex items-center space-x-3">
                <Globe class="h-5 w-5 text-accent" />
                <p>www.pibhyo.org</p>
              </div>
              <div class="flex items-start space-x-3">
                <MapPin class="h-5 w-5 mt-1 text-accent" />
                <p>Av. Leandra Torres 263<br>Huancayo, Perú</p>
              </div>
            </div>
          </div>
          
          <div>
            <h3 class="text-2xl font-bold mb-6 text-accent">Horarios de Culto</h3>
            <div class="space-y-4 opacity-90">
              <div class="flex justify-between items-center">
                <span class="font-medium">Domingos:</span>
                <span>10:00 AM y 6:00 PM</span>
              </div>
              <div class="flex justify-between items-center">
                <span class="font-medium">Miércoles:</span>
                <span>7:00 PM</span>
              </div>
              <div class="bg-accent/20 rounded-lg p-3 mt-4">
                <p class="font-semibold text-center text-accent">
                  Evento Especial: Sept 4-7, 2025
                </p>
              </div>
            </div>
          </div>
        </div>
        
        <div class="border-t border-primary-foreground/20 mt-12 pt-8 text-center">
          <p class="opacity-75 text-lg">
            &copy; 2025 Primera Iglesia Bautista de Huancayo. Todos los derechos reservados.
          </p>
          <p class="font-semibold mt-2 italic text-accent">
            "A Él sea la gloria en la iglesia y en Cristo Jesús" - Efesios 3:21
          </p>
        </div>
      </div>
    </footer>
  </div>
</template>


<style scoped>
html {
  scroll-behavior: smooth;
}

.animate-spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.group:hover .group-hover\:scale-105 {
  transform: scale(1.05);
}

.group:hover .group-hover\:scale-x-100 {
  transform: scaleX(1);
}

input:focus, select:focus, textarea:focus {
  transform: translateY(-1px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
}

::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f5f9;
}

::-webkit-scrollbar-thumb {
  background: #475569;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #334155;
}
</style>
