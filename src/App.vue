<script lang="js" setup>
document.documentElement.lang = 'es-PE';
import { ref, onMounted } from 'vue'
import { Menu, X, MapPin, Calendar, Clock, Users, Phone, Mail, ChevronDown, Star, Award, BookOpen, Quote } from 'lucide-vue-next'
import bgEvent from './assets/images/conference2.png'
import logo from './assets/images/logo/logo-web.webp'
import logo2 from './assets/images/logo/prueba.webp'
import {scheduleData,dayLabels,pastores} from './data'



// Estados para el formulario
const formData = ref({
  nombre: '',
  apellido: '',
  telefono: '',
  iglesia: '',
  funcion: '',
  fechaSalida: '',
  agenciaViaje: ''
})

const formErrors = ref({})
const isSubmitting = ref(false)
const submitMessage = ref('')

// Validaciones
const validateForm = () => {
  const errors = {}
  
  // Validar nombre
  if (!formData.value.nombre.trim()) {
    errors.nombre = 'El nombre es requerido'
  } else if (formData.value.nombre.trim().length < 2) {
    errors.nombre = 'El nombre debe tener al menos 2 caracteres'
  }
  
  // Validar apellido
  if (!formData.value.apellido.trim()) {
    errors.apellido = 'El apellido es requerido'
  } else if (formData.value.apellido.trim().length < 2) {
    errors.apellido = 'El apellido debe tener al menos 2 caracteres'
  }
  
  // Validar teléfono
  const phoneRegex = /^\+?51\s?9\d{8}$/
  if (!formData.value.telefono.trim()) {
    errors.telefono = 'El teléfono es requerido'
  } else if (!phoneRegex.test(formData.value.telefono.replace(/\s/g, ''))) {
    errors.telefono = 'Formato: +51 999 999 999'
  }
  
  // Validar iglesia
  if (!formData.value.iglesia.trim()) {
    errors.iglesia = 'El nombre de la iglesia es requerido'
  }
  
  // Validar función
  if (!formData.value.funcion) {
    errors.funcion = 'Debe seleccionar su función en la iglesia'
  }
  
  // Validar fecha
  if (!formData.value.fechaSalida) {
    errors.fechaSalida = 'La fecha de salida es requerida'
  } else {
    const selectedDate = new Date(formData.value.fechaSalida)
    const eventDate = new Date('2025-09-04')
    if (selectedDate > eventDate) {
      errors.fechaSalida = 'La fecha debe ser antes del evento'
    }
  }
  
  formErrors.value = errors
  return Object.keys(errors).length === 0
}

// Enviar a Google Sheets
const submitForm = async () => {
  if (!validateForm()) {
    submitMessage.value = 'Por favor, corrija los errores en el formulario'
    return
  }
  
  isSubmitting.value = true
  submitMessage.value = ''
  
  try {
    // Crear un FormData para enviar los datos (diferente nombre)
    const form = new FormData()
    form.append('nombre', formData.value.nombre)
    form.append('apellido', formData.value.apellido)
    form.append('telefono', formData.value.telefono)
    form.append('iglesia', formData.value.iglesia)
    form.append('funcion', formData.value.funcion)
    form.append('fechaSalida', formData.value.fechaSalida)
    form.append('agenciaViaje', formData.value.agenciaViaje)
    form.append('fechaRegistro', new Date().toISOString())
    
    // Enviar con fetch
    await fetch('https://script.google.com/macros/s/AKfycbwOtXcY0IBLHQJWA9S-3xGeQwwJemSW1OIKlZ6sWtktfToQRU1Q_WnfYOA-TShW7vEh/exec', {
      method: 'POST',
      mode: 'no-cors',
      body: form
    })
    
    // Mostrar mensaje de éxito
    submitMessage.value = '¡Registro exitoso! Nos pondremos en contacto contigo pronto.'
    
    // Limpiar formulario
    formData.value = {
      nombre: '',
      apellido: '',
      telefono: '',
      iglesia: '',
      funcion: '',
      fechaSalida: '',
      agenciaViaje: ''
    }
    formErrors.value = {}
    
  } catch (error) {
    submitMessage.value = 'Error al enviar el registro. Por favor, intente nuevamente.'
    console.error('Error:', error)
  } finally {
    isSubmitting.value = false
  }
}
// Estado del menú móvil
const mobileMenuOpen = ref(false)

// Estado para los horarios
const activeDay = ref('jueves')
const expandedSchedule = ref(null)

// Contador regresivo
const eventDate = new Date('2025-09-04T10:00:00')
const now = ref(new Date())
const timeLeft = ref({ days: 0, hours: 0, minutes: 0, seconds: 0 })

const updateCountdown = () => {
  const diff = eventDate.getTime() - now.value.getTime()
  if (diff > 0) {
    timeLeft.value = {
      days: Math.floor(diff / (1000 * 60 * 60 * 24)),
      hours: Math.floor((diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)),
      minutes: Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60)),
      seconds: Math.floor((diff % (1000 * 60)) / 1000)
    }
  }
}



onMounted(() => {
  updateCountdown()
  setInterval(() => {
    now.value = new Date()
    updateCountdown()
  }, 1000)
})

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
</script>

<template>
  <div class="min-h-screen bg-background">
    <!-- Navegación mejorada con sombra más sutil -->
    <nav class=" shadow-md border-b border-border sticky top-0 z-50 backdrop-blur-sm bg-background/95">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="flex justify-between items-center h-20">
          <div class="flex-shrink-0">
            <img :src="logo2" alt="Logo Iglesia Bautista" class="h-16 w-auto" />
          </div>
                    
          <div class="hidden md:block">
            <div class="ml-10 flex items-baseline space-x-8">
              <a @click="scrollToSection('inicio')"
                  class="text-foreground-muted hover:text-foreground cursor-pointer px-4 py-2 text-sm font-semibold transition-all duration-300 border-b-2 border-transparent hover:border-foreground relative group">
                Inicio
                <span class="absolute inset-x-0 bottom-0 h-0.5 bg-foreground transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300"></span>
              </a>
              <a @click="scrollToSection('horario')"
                  class="text-foreground-muted hover:text-foreground cursor-pointer px-4 py-2 text-sm font-semibold transition-all duration-300 border-b-2 border-transparent hover:border-foreground relative group">
                Horario
                <span class="absolute inset-x-0 bottom-0 h-0.5 bg-foreground transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300"></span>
              </a>
              <a @click="scrollToSection('predicadores')"
                  class="text-foreground-muted hover:text-foreground cursor-pointer px-4 py-2 text-sm font-semibold transition-all duration-300 border-b-2 border-transparent hover:border-foreground relative group">
                Predicadores
                <span class="absolute inset-x-0 bottom-0 h-0.5 bg-foreground transform scale-x-0 group-hover:scale-x-100 transition-transform duration-300"></span>
              </a>
              <a @click="scrollToSection('registro')"
                  class="btn-primary shadow-md hover:shadow-lg transform hover:-translate-y-0.5 transition-all duration-300">
                Registro
              </a>
            </div>
          </div>
                    
          <div class="md:hidden">
            <button @click="toggleMobileMenu" class="text-foreground-muted hover:text-foreground p-2 rounded-md hover:bg-accent transition-colors">
              <Menu v-if="!mobileMenuOpen" class="h-6 w-6" />
              <X v-else class="h-6 w-6" />
            </button>
          </div>
        </div>
      </div>
            
      <div v-if="mobileMenuOpen" class="md:hidden bg-background/95 backdrop-blur-sm border-t border-border shadow-lg">
        <div class="px-4 pt-4 pb-6 space-y-2">
          <a @click="scrollToSection('inicio')"
              class="block px-4 py-3 text-base font-semibold text-foreground-muted hover:text-foreground hover:bg-accent rounded-md cursor-pointer transition-colors">
            Inicio
          </a>
          <a @click="scrollToSection('horario')"
              class="block px-4 py-3 text-base font-semibold text-foreground-muted hover:text-foreground hover:bg-accent rounded-md cursor-pointer transition-colors">
            Horario
          </a>
          <a @click="scrollToSection('predicadores')"
              class="block px-4 py-3 text-base font-semibold text-foreground-muted hover:text-foreground hover:bg-accent rounded-md cursor-pointer transition-colors">
            Predicadores
          </a>
          <a @click="scrollToSection('registro')"
              class="block mx-4 mt-4 btn-primary text-center">
            Registro
          </a>
        </div>
      </div>
    </nav>

    <!-- Hero Section con mejoras tipográficas -->
    <section id="inicio" class="relative min-h-screen flex items-center justify-center">
      <div class="absolute inset-0 z-0">
        <img :src="bgEvent" alt="Días de Predicación Misionera"
              class="w-full h-full object-cover" />
        <div class="absolute inset-0 bg-gradient-to-b from-primary/80 via-primary/70 to-primary/85"></div>
      </div>
            
      <div class="relative z-10 text-center text-primary-foreground px-4 max-w-5xl mx-auto">
        <div class="mb-12 max-w-4xl mx-auto">
          <h1 class="text-4xl md:text-5xl lg:text-6xl font-serif mb-8 leading-tight font-bold tracking-wide">
            <span class="text-primary-foreground">Días de</span>
            <span class="text-badge block md:inline" style="color: #f2e8d8;"> Predicación </span>
            <span class="text-primary-foreground">Misionera</span>
          </h1>
          
          <div class="bg-primary-foreground/10 backdrop-blur-sm rounded-xl p-8 mb-8 border border-primary-foreground/20 shadow-2xl">
            <Quote class="h-8 w-8 mx-auto mb-4 opacity-60" />
            <p class="text-2xl md:text-3xl lg:text-4xl mb-4 italic font-light leading-relaxed">
              <span class="text-primary-foreground">"Las iglesias eran</span>
              <span class="text-badge font-medium" style="color: #f2e8d8;"> confirmadas </span>
              <span class="text-primary-foreground">en la fe"</span>
            </p>
            <p class="text-xl font-semibold" style="color: #f2e8d8;">Hechos 16:5</p>
          </div>
        </div>
        
        <p class="text-lg mb-8 text-primary-foreground opacity-90 font-medium">4-7 de Septiembre, 2025 • Huancayo, Perú</p>
                
        <div class="flex flex-col sm:flex-row gap-4 justify-center mb-12">
          <button @click="scrollToSection('horario')"
                   class="bg-primary-foreground text-primary px-8 py-4 rounded-md font-semibold text-lg hover:bg-card transition-all duration-300 shadow-lg hover:shadow-xl transform hover:-translate-y-1">
            Ver Horarios
          </button>
          <button @click="scrollToSection('ubicacion')"
                   class="btn-secondary shadow-lg hover:shadow-xl transform hover:-translate-y-1 transition-all duration-300">
            ¿Cómo llegar?
          </button>
        </div>
                
        <div class="bg-primary-foreground/10 backdrop-blur-sm rounded-xl p-8 max-w-2xl mx-auto border border-primary-foreground/20 shadow-2xl">
          <h3 class="text-xl font-semibold mb-6" style="color: #f2e8d8;">Faltan para el evento:</h3>
          <div class="grid grid-cols-4 gap-4">
            <div v-for="(value, key) in timeLeft" :key="key" class="text-center">
              <div class="text-3xl md:text-5xl font-bold text-primary-foreground mb-2">{{ Math.floor(value) }}</div>
              <div class="text-sm uppercase tracking-wide text-primary-foreground opacity-80 font-medium">{{ key }}</div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Sección de versículo destacado -->
    <section class="py-16 bg-primary text-primary-foreground">
      <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
        <Quote class="h-12 w-12 mx-auto mb-6 opacity-60" />
        <p class="text-2xl md:text-3xl font-serif italic leading-relaxed mb-4">
          "Porque no me avergüenzo del evangelio, porque es poder de Dios para salvación a todo aquel que cree."
        </p>
        <p class="text-lg font-semibold opacity-90">Romanos 1:16</p>
      </div>
    </section>

    <!-- Información del evento mejorada -->
    <section class="py-20 bg-accent">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold heading-primary">
            Un tiempo de edificación espiritual
          </h2>
          <p class="text-xl text-foreground-muted max-w-4xl mx-auto leading-relaxed">
            Únete a nosotros del 4 al 7 de septiembre en Av. Leandra Torres 263, Huancayo, 
            para estos días especiales donde la Palabra de Dios será proclamada con poder y fidelidad.
          </p>
        </div>
        
        <div class="grid md:grid-cols-3 gap-8">
          <div class="text-center p-8 bg-card rounded-xl shadow-lg border border-border hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <Calendar class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground">Fechas</h3>
            <p class="text-foreground-muted text-lg">4-7 de Septiembre, 2025</p>
            <p class="text-sm text-foreground-muted mt-2">Cuatro días de bendición</p>
          </div>
          
          <div class="text-center p-8 bg-card rounded-xl shadow-lg border border-border hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2">
            <div class="bg-primary rounded-full w-16 h-16 flex items-center justify-center mx-auto mb-6 shadow-md">
              <Users class="h-8 w-8 text-primary-foreground" />
            </div>
            <h3 class="text-2xl font-bold mb-4 text-foreground">Predicador Principal</h3>
            <p class="text-foreground-muted text-lg font-semibold">Hno. Daniel Rose</p>
            <p class="text-sm text-foreground-muted mt-2">Ministerio Internacional</p>
          </div>
          
          <div class="text-center p-8 bg-card rounded-xl shadow-lg border border-border hover:shadow-xl transition-all duration-300 transform hover:-translate-y-2">
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

    <!-- Sección de horarios mejorada -->
    <section id="horario" class="py-20 bg-background">
      <div class="max-w-5xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold heading-primary">
            Horario del Evento
          </h2>
          <p class="text-xl text-foreground-muted">Predicador principal: <strong class="text-foreground">Hno. Daniel Rose</strong></p>
        </div>
                
        <div class="flex flex-wrap justify-center mb-12 bg-accent rounded-xl p-2 max-w-2xl mx-auto shadow-inner">
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
            class="bg-card rounded-xl border-2 border-border overflow-hidden hover:border-foreground-muted transition-all duration-300 shadow-md hover:shadow-lg"
          >
            <div 
               @click="toggleScheduleExpand(item.time)"
              class="flex items-center justify-between p-8 cursor-pointer hover:bg-accent transition-colors duration-200"
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
              class="px-8 pb-8 text-foreground-muted border-t border-border bg-accent/50"
            >
              <p class="mt-6 text-lg leading-relaxed">{{ item.description }}</p>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Sección de predicadores MEJORADA con imágenes -->
    <section id="predicadores" class="py-20 bg-accent">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold heading-primary">
            Predicadores y Pastores
          </h2>
          <p class="text-xl text-foreground-muted max-w-3xl mx-auto leading-relaxed">
            Siervos de Dios comprometidos con la predicación fiel de Su Palabra y la edificación del pueblo del Señor
          </p>
        </div>
                
        <div class="grid lg:grid-cols-3 gap-10">
          <div v-for="pastor in pastores" :key="pastor.nombre" 
               class="bg-card rounded-2xl shadow-xl border border-border hover:shadow-2xl transition-all duration-500 transform hover:-translate-y-3 overflow-hidden group">
            
            <!-- Imagen del pastor -->
            <div class="relative overflow-hidden">
              <img :src="pastor.imagen" :alt="pastor.nombre"
                   class="w-full h-80 object-cover transition-transform duration-500 group-hover:scale-105" />
              <div class="absolute inset-0 bg-gradient-to-t from-primary/20 to-transparent opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
            </div>
            
            <!-- Contenido de la tarjeta -->
            <div class="p-8">
              <div class="text-center mb-6">
                <h3 class="text-2xl font-bold text-foreground mb-2">{{ pastor.nombre }}</h3>
                <p class="text-foreground-muted font-semibold text-lg mb-1">{{ pastor.titulo }}</p>
                <div class="flex justify-center space-x-4 text-sm text-foreground-muted">
                  <span class="bg-badge px-3 py-1 rounded-full">{{ pastor.ministerio }}</span>
                  <span class="bg-badge px-3 py-1 rounded-full">{{ pastor.experiencia }}</span>
                </div>
              </div>
              
              <p class="text-foreground-muted leading-relaxed mb-6 text-center">{{ pastor.descripcion }}</p>
              
              <div class="text-center">
                <div class="bg-primary/10 rounded-lg px-4 py-3 inline-block border border-primary/20">
                  <div class="flex items-center justify-center space-x-2">
                    <BookOpen class="h-4 w-4 text-foreground" />
                    <span class="text-foreground font-semibold text-sm">{{ pastor.especialidad }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- Sección adicional de compromiso ministerial -->
        <div class="mt-16 bg-primary/5 rounded-2xl p-8 border border-primary/10">
          <div class="text-center">
            <Quote class="h-10 w-10 mx-auto mb-4 text-foreground-muted" />
            <p class="text-xl font-serif italic text-foreground mb-4 leading-relaxed">
              "Id por todo el mundo y predicad el evangelio a toda criatura."
            </p>
            <p class="text-foreground-muted font-semibold">Marcos 16:15</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Sección de ubicación mejorada -->
    <section id="ubicacion" class="py-20 bg-background">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-16">
          <h2 class="text-4xl md:text-5xl font-serif text-foreground mb-6 font-bold heading-primary">
            Ubicación
          </h2>
          <p class="text-xl text-foreground-muted">Te esperamos en nuestra iglesia</p>
        </div>
                
        <div class="grid lg:grid-cols-3 gap-12">
          <div class="lg:col-span-1">
            <div class="bg-card p-8 rounded-2xl shadow-xl border border-border h-full">
              <h3 class="text-2xl font-bold mb-8 text-foreground">Información de Contacto</h3>
                            
              <div class="space-y-6">
                <div class="flex items-start space-x-4 p-4 bg-accent/50 rounded-lg">
                  <div class="bg-primary rounded-full w-12 h-12 flex items-center justify-center flex-shrink-0">
                    <MapPin class="h-6 w-6 text-primary-foreground" />
                  </div>
                  <div>
                    <p class="font-bold text-foreground mb-1">Dirección</p>
                    <p class="text-foreground-muted">Av. Leandra Torres 263<br>Huancayo, Perú</p>
                  </div>
                </div>
                                
                <div class="flex items-start space-x-4 p-4 bg-accent/50 rounded-lg">
                  <div class="bg-primary rounded-full w-12 h-12 flex items-center justify-center flex-shrink-0">
                    <Phone class="h-6 w-6 text-primary-foreground" />
                  </div>
                  <div>
                    <p class="font-bold text-foreground mb-1">Teléfono</p>
                    <p class="text-foreground-muted">+51 999 999 999</p>
                  </div>
                </div>
                                
                <div class="flex items-start space-x-4 p-4 bg-accent/50 rounded-lg">
                  <div class="bg-primary rounded-full w-12 h-12 flex items-center justify-center flex-shrink-0">
                    <Mail class="h-6 w-6 text-primary-foreground" />
                  </div>
                  <div>
                    <p class="font-bold text-foreground mb-1">Email</p>
                    <p class="text-foreground-muted">contacto@iglesia.com</p>
                  </div>
                </div>
              </div>
                            
              <div class="mt-8 pt-8 border-t border-border">
                <h4 class="font-bold text-foreground mb-4 text-lg">Horarios de Culto</h4>
                <div class="space-y-3 text-foreground-muted">
                  <div class="flex justify-between items-center p-2 rounded">
                    <span class="font-medium">Domingos</span>
                    <span>10:00 AM - 6:00 PM</span>
                  </div>
                  <div class="flex justify-between items-center p-2 rounded">
                    <span class="font-medium">Miércoles</span>
                    <span>7:00 PM</span>
                  </div>
                  <div class="flex justify-between items-center bg-badge px-3 py-2 rounded-md mb-8">
                    <span class="font-medium text-badge-foreground">Evento Especial</span>
                    <span class="text-badge-foreground font-semibold">Sept 4-7</span>
                  </div>
                </div>
              </div>
                            
              <a  class="btn-primary w-full mt-8 shadow-lg hover:shadow-xl transform hover:-translate-y-1 transition-all duration-300" target="_blank" href="https://maps.app.goo.gl/43H5Q16MrWckdVks6">
                Ver en Google Maps
              </a>
            </div>
          </div>
                    
          <!-- Mapa Real de Google Maps -->
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

    <!-- Sección de registro mejorada -->
    <section id="registro" class="py-20 bg-primary text-primary-foreground">
      <div class="max-w-4xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="text-center mb-12">
          <h2 class="text-4xl md:text-5xl font-serif mb-6 font-bold">
            Registro
          </h2>
          <p class="text-xl mb-4 opacity-90">
            Confirma tu asistencia a estos días especiales
          </p>
          <p class="text-lg opacity-75">
            4-7 de Septiembre • Av. Leandra Torres 263, Huancayo
          </p>
        </div>
                
        <div class="max-w-3xl mx-auto bg-primary-foreground/10 backdrop-blur-sm rounded-2xl p-10 border border-primary-foreground/20 shadow-2xl">
          <form @submit.prevent="submitForm" class="space-y-8">
   Información personal 
  <div class="border-b border-primary-foreground/20 pb-8">
    <h3 class="text-2xl font-bold mb-6" style="color: #f2e8d8;">Información Personal</h3>
    <div class="grid md:grid-cols-2 gap-6">
      <div>
        <label class="block text-sm font-semibold mb-3 opacity-90">Nombre *</label>
        <input 
          type="text" 
          v-model="formData.nombre"
          placeholder="Tu nombre"
          :class="[
            'w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
            formErrors.nombre ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
          ]"
        >
        <p v-if="formErrors.nombre" class="text-red-400 text-sm mt-2">{{ formErrors.nombre }}</p>
      </div>
      <div>
        <label class="block text-sm font-semibold mb-3 opacity-90">Apellido *</label>
        <input 
          type="text" 
          v-model="formData.apellido"
          placeholder="Tu apellido"
          :class="[
            'w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
            formErrors.apellido ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
          ]"
        >
        <p v-if="formErrors.apellido" class="text-red-400 text-sm mt-2">{{ formErrors.apellido }}</p>
      </div>
    </div>
    <div class="mt-6">
      <label class="block text-sm font-semibold mb-3 opacity-90">Número de teléfono *</label>
      <input 
        type="tel" 
        v-model="formData.telefono"
        placeholder="+51 999 999 999"
        :class="[
          'w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
          formErrors.telefono ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
        ]"
      >
      <p v-if="formErrors.telefono" class="text-red-400 text-sm mt-2">{{ formErrors.telefono }}</p>
    </div>
  </div>
  
   Información ministerial 
  <div class="border-b border-primary-foreground/20 pb-8">
    <h3 class="text-2xl font-bold mb-6" style="color: #f2e8d8;">Información Ministerial</h3>
    <div class="space-y-6">
      <div>
        <label class="block text-sm font-semibold mb-3 opacity-90">Nombre de la iglesia donde congrega *</label>
        <input 
          type="text" 
          v-model="formData.iglesia"
          placeholder="Iglesia Bautista..."
          :class="[
            'w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
            formErrors.iglesia ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
          ]"
        >
        <p v-if="formErrors.iglesia" class="text-red-400 text-sm mt-2">{{ formErrors.iglesia }}</p>
      </div>
      <div>
        <label class="block text-sm font-semibold mb-3 opacity-90">¿Qué función desempeña actualmente en la iglesia? *</label>
        <select 
          v-model="formData.funcion"
          :class="[
            'w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
            formErrors.funcion ? 'border-red-500' : 'border-transparent focus:border-primary-foreground'
          ]"
        >
          <option value="">Selecciona tu función</option>
          <option value="pastor">Pastor</option>
          <option value="anciano">Anciano</option>
          <option value="diacono">Diácono</option>
          <option value="maestro">Maestro/Predicador</option>
          <option value="lider">Líder de ministerio</option>
          <option value="miembro">Miembro activo</option>
          <option value="otro">Otro</option>
        </select>
        <p v-if="formErrors.funcion" class="text-red-400 text-sm mt-2">{{ formErrors.funcion }}</p>
      </div>
    </div>
  </div>
  
   Información de viaje 
  <div class="pb-8">
    <h3 class="text-2xl font-bold mb-6" style="color: #f2e8d8;">Información de Viaje</h3>
    <div class="space-y-6">
      <div>
        <label class="block text-sm font-semibold mb-3 opacity-90">Fecha de salida *</label>
        <input 
          type="date"
          v-model="formData.fechaSalida"
          lang="es"
          :class="[
            'w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2',
            formErrors.fechaSalida ? 'border-red-500' : 'border-transparent focus:border-primary-foreground' 
          ]"
        >
        <p v-if="formErrors.fechaSalida" class="text-red-400 text-sm mt-2">{{ formErrors.fechaSalida }}</p>
      </div>
      <div>
        <label class="block text-sm font-semibold mb-3 opacity-90">Agencia donde piensa viajar (opcional)</label>
        <input 
          type="text" 
          v-model="formData.agenciaViaje"
          placeholder="Nombre de la agencia de viajes"
          class="w-full px-4 py-4 rounded-lg bg-primary-foreground text-primary placeholder-foreground-muted focus:outline-none focus:ring-4 focus:ring-primary-foreground/50 transition-all text-lg border-2 border-transparent focus:border-primary-foreground"
        >
      </div>
    </div>
  </div>
  
   Mensaje de estado 
  <div v-if="submitMessage" :class="[
    'text-center p-4 rounded-lg',
    submitMessage.includes('exitoso') ? 'bg-green-500/20 text-green-300' : 'bg-red-500/20 text-red-300'
  ]">
    {{ submitMessage }}
  </div>
  
  <button 
    type="submit"
    :disabled="isSubmitting"
    :class="[
      'w-full px-8 py-5 rounded-lg font-bold text-xl transition-all duration-300 shadow-lg hover:shadow-xl transform',
      isSubmitting 
        ? 'bg-gray-400 text-gray-600 cursor-not-allowed' 
        : 'bg-primary-foreground text-primary hover:bg-card hover:-translate-y-1'
    ]"
  >
    {{ isSubmitting ? 'Enviando...' : 'Confirmar Registro' }}
  </button>
</form>
                    
          <div class="mt-10 pt-8 border-t border-primary-foreground/20 text-center">
            <p class="opacity-75 mb-3">¿Tienes preguntas?</p>
            <p class="font-bold text-xl" style="color: #f2e8d8;">+51 999 999 999</p>
          </div>
        </div>
      </div>
    </section>

    <!-- Footer mejorado -->
    <footer class="bg-foreground text-primary-foreground py-16 border-t-4 section-highlight">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div class="grid md:grid-cols-3 gap-12">
          <div>
            <img :src="logo" alt="Logo Iglesia Bautista" class="h-16 w-auto mb-6 filter brightness-0 invert" />
            <p class="opacity-90 leading-relaxed text-lg">
              Proclamando fielmente la Palabra de Dios para la gloria de Cristo y la edificación de Su iglesia.
            </p>
            <p class="font-semibold mt-4 italic" style="color: #f2e8d8;">"Soli Deo Gloria"</p>
          </div>
                    
          <div>
            <h3 class="text-2xl font-bold mb-6" style="color: #f2e8d8;">Contacto</h3>
            <div class="space-y-4 opacity-90">
              <div class="flex items-center space-x-3">
                <Phone class="h-5 w-5" style="color: #f2e8d8;" />
                <p>+51 999 999 999</p>
              </div>
              <div class="flex items-center space-x-3">
                <Mail class="h-5 w-5" style="color: #f2e8d8;" />
                <p>contacto@iglesia.com</p>
              </div>
              <div class="flex items-start space-x-3">
                <MapPin class="h-5 w-5 mt-1" style="color: #f2e8d8;" />
                <p>Av. Leandra Torres 263<br>Huancayo, Perú</p>
              </div>
            </div>
          </div>
                    
          <div>
            <h3 class="text-2xl font-bold mb-6" style="color: #f2e8d8;">Horarios de Culto</h3>
            <div class="space-y-4 opacity-90">
              <div class="flex justify-between items-center">
                <span class="font-medium">Domingos:</span>
                <span>10:00 AM y 6:00 PM</span>
              </div>
              <div class="flex justify-between items-center">
                <span class="font-medium">Miércoles:</span>
                <span>7:00 PM</span>
              </div>
              <div class="bg-badge/20 rounded-lg p-3 mt-4">
                <p class="font-semibold text-center" style="color: #f2e8d8;">
                  Evento Especial: Sept 4-7, 2025
                </p>
              </div>
            </div>
          </div>
        </div>
                
        <div class="border-t border-primary-foreground/20 mt-12 pt-8 text-center">
          <p class="opacity-75 text-lg">
            &copy; 2025 Iglesia Bautista. Todos los derechos reservados.
          </p>
          <p class="font-semibold mt-2 italic" style="color: #f2e8d8;">
            "A Él sea la gloria en la iglesia y en Cristo Jesús" - Efesios 3:21
          </p>
        </div>
      </div>
    </footer>
  </div>
</template>

<style scoped>
@import "tailwindcss";

/* Esquema conservador para ministerio establecido - Inspirado en Grace To You */
@theme {
  /* Superficies */
  --color-background: #fff8ed; /* fondo base (claro y cálido) */
  --color-card: #f8f8f8; /* tarjetas, bloques neutros */
  --color-accent: #e9e8e0; /* detalles sutiles, separadores */
  --color-border: #e9e8e0; /* bordes y divisores */
  
  /* Texto */
  --color-foreground: #232c32; /* texto principal en fondos claros */
  --color-foreground-muted: #333333; /* texto secundario */
  
  /* CTA / acciones */
  --color-primary: #232c32; /* botón principal (oscuro) */
  --color-primary-foreground: #f8f8f8; /* texto dentro del CTA */
  --color-primary-hover: #293339; /* hover del CTA */
  
  /* Opcional: si quieres etiquetas o chips tenues sobre claro */
  --color-badge: #f2e8d8;
  --color-badge-foreground: #232c32;
}

/* Clases de utilidad conservadoras */
@layer utilities {
  .text-primary {
    color: var(--color-foreground);
  }
  .bg-background {
    background-color: var(--color-background);
  }
  .bg-primary {
    background-color: var(--color-primary);
    color: var(--color-primary-foreground);
  }
  .text-muted {
    color: var(--color-foreground-muted);
  }
  
  /* Botón principal conservador */
  .btn-primary {
    cursor: pointer;
    background-color: var(--color-primary);
    color: var(--color-primary-foreground);
    padding: 0.75rem 2rem;
    border-radius: 0.375rem;
    font-weight: 600;
    font-size: 0.875rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    transition: all 0.3s ease;
    border: 2px solid var(--color-primary);
  }
  .btn-primary:hover {
    background-color: var(--color-primary-hover);
    border-color: var(--color-primary-hover);
  }
  
  /* Botón secundario */
  .btn-secondary {
    cursor: pointer;
    background-color: transparent;
    color: var(--color-primary-foreground);
    padding: 0.75rem 2rem;
    border: 2px solid var(--color-primary-foreground);
    border-radius: 0.375rem;
    font-weight: 600;
    font-size: 0.875rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    transition: all 0.3s ease;
  }
  .btn-secondary:hover {
    background-color: var(--color-primary-foreground);
    color: var(--color-primary);
  }
  
  /* Encabezados conservadores */
  .heading-primary {
    color: var(--color-primary);
    font-weight: 700;
    line-height: 1.2;
  }
  
  /* Secciones destacadas */
  .section-highlight {
    border-left: 4px solid var(--color-primary);
  }
}

/* Tipografía conservadora */
.font-serif {
  font-family: "Georgia", "Times New Roman", serif;
}

/* Transiciones suaves mejoradas */
* {
  transition: all 0.3s ease;
}

/* Scroll suave */
html {
  scroll-behavior: smooth;
}

/* Animación para el marcador del mapa */
@keyframes pulse {
  0%, 100% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.1);
    opacity: 0.8;
  }
}

.animate-pulse {
  animation: pulse 2s infinite;
}

/* Mejoras en los inputs del formulario */
input:focus, select:focus {
  transform: translateY(-2px);
  box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
}

/* Hover effects mejorados para las tarjetas */
.hover\:shadow-xl:hover {
  transform: translateY(-4px);
}

/* Efectos de navegación mejorados */
.group:hover .group-hover\:scale-x-100 {
  transform: scaleX(1);
}

/* Mejoras en las transiciones de las imágenes */
img {
  transition: transform 0.5s ease;
}
</style>
