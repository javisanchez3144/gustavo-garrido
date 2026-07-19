<script setup>
import { computed, nextTick, onMounted, reactive, ref } from 'vue';
import heroImage from './assets/hero-vascular.png';

const heroDoctorImage = '/Dr2.jpeg';

const contact = {
  doctor: 'Dr. Gustavo Garrido',
  specialty: 'Cirugia Cardiovascular, Vascular y Endovascular',
  city: 'Portoviejo, Manabi',
  phone: '0987449509',
  phoneAlt: '0969264000',
  whatsapp: '593987449509',
  instagram: '@drgustavogarrido',
  instagramUrl: 'https://www.instagram.com/drgustavogarrido/',
  email: 'citas@drgustavogarrido.com',
  address: 'Consultorio medico en Portoviejo',
  mapUrl: 'https://maps.app.goo.gl/1tDrKnvxBWwZAnb29',
  mapEmbedUrl: 'https://maps.google.com/maps?q=-1.023277,-80.475687&z=17&output=embed'
};

const form = reactive({
  name: '',
  phone: '',
  reason: 'Valoracion vascular',
  date: '',
  message: ''
});

const vascularRisk = reactive({
  age: '',
  diabetes: false,
  hypertension: false,
  smoker: false,
  cholesterol: false,
  symptoms: []
});

const today = new Date().toISOString().split('T')[0];
const sent = ref(false);
const isLoading = ref(true);
const activeSection = ref('inicio');
const activeVideo = ref(0);
const activeGallery = ref(0);
const activeTestimonial = ref(0);
const activeArticle = ref(-1);
const activeProcedure = ref(0);
const procedureCarousel = ref(null);
const fullHeroTitle = 'Atención vascular especializada';
const typedHeroTitle = ref('');
const isTypingHeroTitle = ref(false);
const doctorPhotoAvailable = ref(true);
const logoSrc = '/logo-dr.jpg';
const doctorPhotoSrc = '/dr-gustavo-garrido.jpg';

const whatsappUrl = computed(() => {
  const text = [
    `Hola, deseo agendar una cita con ${contact.doctor}.`,
    `Nombre: ${form.name || 'Por completar'}`,
    `Telefono: ${form.phone || 'Por completar'}`,
    `Motivo: ${form.reason}`,
    `Fecha preferida: ${form.date || 'Por coordinar'}`,
    form.message ? `Mensaje: ${form.message}` : ''
  ].filter(Boolean).join('\n');

  return `https://wa.me/${contact.whatsapp}?text=${encodeURIComponent(text)}`;
});

const riskSymptoms = [
  { value: 'dolorCaminar', label: 'Dolor en piernas al caminar', points: 3 },
  { value: 'heridas', label: 'Heridas que no cicatrizan', points: 4 },
  { value: 'hinchazon', label: 'Hinchazon persistente', points: 2 },
  { value: 'frioColor', label: 'Pie frio o cambio de color', points: 4 },
  { value: 'varicesDolor', label: 'Varices con dolor o pesadez', points: 2 },
  { value: 'dolorReposo', label: 'Dolor en reposo', points: 5 }
];

const calculatedAge = computed(() => {
  return Number(vascularRisk.age) || 0;
});

const vascularRiskResult = computed(() => {
  let score = 0;
  const age = calculatedAge.value;

  if (age >= 65) score += 3;
  else if (age >= 50) score += 2;
  else if (age >= 40) score += 1;

  if (vascularRisk.diabetes) score += 3;
  if (vascularRisk.hypertension) score += 2;
  if (vascularRisk.smoker) score += 2;
  if (vascularRisk.cholesterol) score += 1;

  vascularRisk.symptoms.forEach((selected) => {
    const symptom = riskSymptoms.find((item) => item.value === selected);
    if (symptom) score += symptom.points;
  });

  const urgent = vascularRisk.symptoms.some((symptom) => ['heridas', 'frioColor', 'dolorReposo'].includes(symptom));

  if (urgent || score >= 8) {
    return {
      level: 'Riesgo alto',
      className: 'high',
      score,
      message: 'Conviene solicitar una valoracion vascular pronto, especialmente si los sintomas son recientes, progresivos o afectan un solo pie o pierna.',
      action: 'Solicitar valoracion'
    };
  }

  if (score >= 4) {
    return {
      level: 'Riesgo moderado',
      className: 'moderate',
      score,
      message: 'Hay factores que justifican una revision preventiva y seguimiento de sintomas.',
      action: 'Consultar por prevencion'
    };
  }

  return {
    level: 'Riesgo bajo',
    className: 'low',
    score,
    message: 'No se identifican señales fuertes en este cuestionario, pero si las molestias persisten es recomendable consultar.',
    action: 'Resolver dudas'
  };
});

const quickAreas = [
  'Varices',
  'Pie diabetico',
  'Trombosis venosa',
  'Aneurisma de aorta',
  'Enfermedad arterial periferica',
  'Ulceras venosas',
  'Linfedema',
  'Accesos para hemodialisis'
];

const conditions = [
  {
    title: 'Varices',
    imageClass: 'varices',
    summary: 'Venas dilatadas que pueden causar pesadez, ardor, calambres, cambios en la piel o ulceras.',
    symptoms: 'Dolor, hinchazon, venas visibles, cansancio al final del dia.',
    care: 'Eco Doppler, manejo medico, escleroterapia, laser, radiofrecuencia o cirugia segun el caso.'
  },
  {
    title: 'Pie diabetico',
    imageClass: 'diabetic-foot',
    summary: 'Heridas, infecciones o mala circulacion en pacientes con diabetes que requieren control temprano.',
    symptoms: 'Heridas que no cierran, cambios de color, dolor, secrecion, piel fria o perdida de sensibilidad.',
    care: 'Evaluacion vascular, curaciones, control de infeccion y estrategias para prevenir amputaciones.'
  },
  {
    title: 'Trombosis venosa profunda',
    imageClass: 'thrombosis',
    summary: 'Coagulos en venas profundas que pueden producir complicaciones si no se diagnostican a tiempo.',
    symptoms: 'Pierna hinchada, dolor de pantorrilla, calor local o cambio brusco de volumen.',
    care: 'Diagnostico con Doppler, anticoagulacion y seguimiento especializado.'
  },
  {
    title: 'Enfermedad arterial periferica',
    imageClass: 'arterial',
    summary: 'Disminucion del flujo arterial hacia piernas y pies, frecuente en diabetes, tabaquismo o hipertension.',
    symptoms: 'Dolor al caminar, pies frios, heridas persistentes o dolor en reposo.',
    care: 'Estudios vasculares, control de riesgo, angioplastia, stents o cirugia arterial.'
  },
  {
    title: 'Aneurisma de aorta',
    imageClass: 'aneurysm',
    summary: 'Dilatacion de una arteria principal que suele avanzar sin sintomas y necesita vigilancia.',
    symptoms: 'Puede no dar molestias; a veces dolor abdominal, lumbar o hallazgo en estudios.',
    care: 'Seguimiento por imagen, control de riesgo y tratamiento endovascular o quirurgico cuando corresponde.'
  },
  {
    title: 'Ulceras venosas y linfedema',
    imageClass: 'ulcers',
    summary: 'Heridas cronicas o hinchazon persistente que afectan movilidad, piel y calidad de vida.',
    symptoms: 'Piernas pesadas, piel endurecida, liquido, heridas recurrentes o infecciones.',
    care: 'Doppler, compresion, curaciones avanzadas y plan de mantenimiento.'
  }
];

const credentials = [
  'Cirugia cardiovascular, vascular y endovascular',
  'Flebologia y linfologia',
  'Mas de 20 anos de experiencia',
  'Atencion de enfermedades venosas, arteriales y pie diabetico',
  'Participacion academica y actualizacion continua',
  'Filosofia centrada en explicar, prevenir y acompanar'
];

const procedures = [
  {
    title: 'Eco Doppler vascular',
    tag: 'Diagnóstico',
    description: 'Estudio no invasivo para evaluar flujo arterial y venoso, detectar trombosis, reflujo venoso o alteraciones de circulación.'
  },
  {
    title: 'Cirugía de varices',
    tag: 'Tratamiento venoso',
    description: 'Opciones para tratar venas enfermas cuando causan dolor, pesadez, inflamación, cambios en la piel o riesgo de complicaciones.'
  },
  {
    title: 'Radiofrecuencia y láser venoso',
    tag: 'Mínima invasión',
    description: 'Técnicas modernas que cierran venas con reflujo mediante calor controlado, con incisiones pequeñas y recuperación más cómoda.'
  },
  {
    title: 'Escleroterapia',
    tag: 'Venas visibles',
    description: 'Tratamiento para arañas vasculares y algunas venas superficiales mediante aplicación de una sustancia que ayuda a cerrar la vena.'
  },
  {
    title: 'Angioplastias y stents',
    tag: 'Endovascular',
    description: 'Procedimientos para abrir arterias estrechas u obstruidas y mejorar el flujo sanguíneo hacia piernas, pies u otros territorios.'
  },
  {
    title: 'Cirugía arterial',
    tag: 'Circulación',
    description: 'Tratamiento quirúrgico para enfermedad arterial avanzada, isquemia, aneurismas o casos donde se necesita restaurar el flujo.'
  },
  {
    title: 'Manejo integral del pie diabético',
    tag: 'Prevención',
    description: 'Evaluación de heridas, infección, sensibilidad y circulación para reducir complicaciones y proteger la extremidad.'
  },
  {
    title: 'Accesos vasculares para hemodiálisis',
    tag: 'Hemodiálisis',
    description: 'Creación y cuidado de accesos vasculares para pacientes que requieren terapia renal, priorizando durabilidad y seguridad.'
  }
];

const articles = [
  {
    title: '¿Las varices siempre necesitan cirugía?',
    description: 'No siempre. Muchas varices se manejan primero con ejercicio, elevación de piernas y medias de compresión. Si hay dolor, pesadez, hinchazón, cambios en la piel o el Doppler muestra reflujo importante, pueden considerarse tratamientos como escleroterapia, láser, radiofrecuencia o cirugía ambulatoria.'
  },
  {
    title: '¿Cuándo una pierna hinchada puede ser peligrosa?',
    description: 'La hinchazón debe valorarse pronto si aparece de forma repentina, afecta una sola pierna, se acompaña de dolor, piel caliente, cambio de color, pie frío o palidez. Si además hay falta de aire, dolor de pecho, mareo o tos con sangre, se debe buscar atención de emergencia.'
  },
  {
    title: '¿Cómo prevenir una amputación por diabetes?',
    description: 'La prevención empieza con buen control de glucosa, revisión diaria de los pies, calzado adecuado y controles médicos periódicos. En diabetes, una herida pequeña puede complicarse si hay pérdida de sensibilidad o mala circulación. Dolor, entumecimiento, cambio de color, hinchazón o una herida que no cierra requieren valoración temprana.'
  },
  {
    title: '¿Qué es una trombosis y cuándo consultar?',
    description: 'La trombosis venosa profunda ocurre cuando se forma un coágulo en una vena profunda, generalmente en la pierna. Puede causar hinchazón, dolor o calambre en la pantorrilla, calor local y cambio de color, aunque a veces no da síntomas. Debe consultarse porque puede complicarse si el coágulo viaja al pulmón.'
  },
  {
    title: 'Dolor al caminar: ¿señal de mala circulación?',
    description: 'Puede ser señal de enfermedad arterial periférica si el dolor o calambre aparece al caminar cierta distancia y mejora al descansar. También pueden aparecer pies fríos, debilidad, adormecimiento o heridas que tardan en cerrar. Es más importante evaluarlo si hay diabetes, hipertensión, colesterol alto o tabaquismo.'
  },
  {
    title: '¿Cómo cuidar una úlcera venosa en casa?',
    description: 'Una úlcera venosa necesita limpieza, apósitos adecuados, elevación de la pierna y compresión cuando el especialista la indica. La compresión debe colocarse correctamente y no se recomienda improvisarla si hay diabetes, dolor inusual, dedos pálidos o azules, hormigueo o sospecha de mala circulación arterial.'
  }
];

const galleryItems = [
  {
    type: 'Diagnostico',
    title: 'Ecografias Doppler',
    detail: 'Imagenes educativas para explicar flujo venoso, arterias, valvulas y hallazgos relevantes durante la consulta.',
    meta: 'Doppler vascular',
    image: '/galeria/1.jpg',
    tone: 'doppler'
  },
  {
    type: 'Procedimientos',
    title: 'Angiografias',
    detail: 'Estudios visuales que ayudan a comprender obstrucciones, circulacion distal y decisiones endovasculares.',
    meta: 'Arterial y endovascular',
    image: '/galeria/2.jpg',
    tone: 'angiography'
  },
  {
    type: 'Evolucion',
    title: 'Antes y despues',
    detail: 'Casos clinicos documentados solo con consentimiento, cuidando privacidad e identidad del paciente.',
    meta: 'Con autorizacion',
    image: '/galeria/3.jpg',
    tone: 'before-after'
  },
  {
    type: 'Video educativo',
    title: 'Capsulas cortas',
    detail: 'Videos de 1 a 3 minutos para explicar varices, pie diabetico, trombosis, aneurismas y Doppler vascular.',
    meta: 'Paciente informado',
    image: '/galeria/4.jpg',
    tone: 'short-video'
  },
  {
    type: 'Material descargable',
    title: 'Infografias',
    detail: 'Guias visuales para cuidados en casa, senales de alarma y preparacion antes de estudios o procedimientos.',
    meta: 'Educacion vascular',
    image: '/galeria/5.jpg',
    tone: 'infographic'
  }
];

const instagramReels = [
  { title: 'Hormigueo y sintomas vasculares', src: '/video1.mp4' },
  { title: 'Cirugia y cuidado vascular', src: '/vide2.mp4' },
  { title: 'Prevencion vascular', src: '/video3.mp4' },
  { title: 'Tratamientos vasculares', src: '/video4.mp4' }
];

const testimonials = [
  {
    title: 'Testimonio en video',
    text: 'Espacio para pacientes que autoricen compartir su experiencia despues de una valoracion o tratamiento.',
    meta: 'Con autorizacion'
  },
  {
    title: 'Historia clinica breve',
    text: 'Relatos centrados en el proceso: sintomas, diagnostico, plan y seguimiento, sin datos sensibles.',
    meta: 'Privacidad cuidada'
  },
  {
    title: 'Confianza del paciente',
    text: 'La voz del paciente ayuda a que otros entiendan que consultar a tiempo cambia decisiones.',
    meta: 'Formato humano'
  }
];

const faq = [
  {
    q: 'Cuando debo acudir a un cirujano vascular?',
    a: 'Cuando hay varices dolorosas, pierna hinchada, heridas que no cicatrizan, dolor al caminar, pie diabetico, sospecha de trombosis o antecedentes de aneurisma.'
  },
  {
    q: 'El formulario confirma automaticamente la cita?',
    a: 'No. Prepara el mensaje para WhatsApp y el equipo confirma disponibilidad, horario y preparacion.'
  },
  {
    q: 'Que estudios debo llevar?',
    a: 'Si tienes examenes, ecografias, angiografias, recetas o informes previos, es recomendable llevarlos a la consulta.'
  },
  {
    q: 'Las tecnicas endovasculares requieren heridas grandes?',
    a: 'Muchas tecnicas endovasculares se realizan por accesos pequenos. La indicacion depende del diagnostico y del estado general del paciente.'
  }
];

function submitAppointment() {
  form.phone = form.phone.replace(/\D/g, '');
  if (form.date && form.date < today) {
    form.date = today;
    return;
  }

  sent.value = true;
  window.open(whatsappUrl.value, '_blank', 'noopener,noreferrer');
}

function scrollToTop() {
  window.scrollTo({ top: 0, behavior: 'smooth' });
}

function nextGallery() {
  activeGallery.value = (activeGallery.value + 1) % galleryItems.length;
}

function previousGallery() {
  activeGallery.value = (activeGallery.value - 1 + galleryItems.length) % galleryItems.length;
}

function selectGallery(index) {
  activeGallery.value = index;
}

function selectProcedure(index) {
  activeProcedure.value = index;
  nextTick(() => {
    const carousel = procedureCarousel.value;
    const card = carousel?.querySelectorAll('.procedure-card')[index];
    if (!carousel || !card) return;

    carousel.scrollTo({
      behavior: 'smooth',
      left: card.offsetLeft - (carousel.clientWidth - card.clientWidth) / 2
    });
  });
}

function nextProcedure() {
  selectProcedure((activeProcedure.value + 1) % procedures.length);
}

function previousProcedure() {
  selectProcedure((activeProcedure.value - 1 + procedures.length) % procedures.length);
}

function typeHeroTitle() {
  typedHeroTitle.value = '';
  isTypingHeroTitle.value = true;

  fullHeroTitle.split('').forEach((letter, index) => {
    window.setTimeout(() => {
      typedHeroTitle.value += letter;
      if (index === fullHeroTitle.length - 1) {
        window.setTimeout(() => {
          isTypingHeroTitle.value = false;
        }, 350);
      }
    }, index * 52);
  });
}

onMounted(() => {
  window.setTimeout(() => {
    isLoading.value = false;
    window.setTimeout(typeHeroTitle, 180);
  }, 500);

  const revealItems = document.querySelectorAll('.reveal');
  const revealObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (!entry.isIntersecting) return;
      entry.target.classList.add('is-visible');
      revealObserver.unobserve(entry.target);
    });
  }, { threshold: 0.15 });

  revealItems.forEach((item) => revealObserver.observe(item));

  const sections = document.querySelectorAll('section[id]');
  const navObserver = new IntersectionObserver((entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) activeSection.value = entry.target.id;
    });
  }, { rootMargin: '-34% 0px -56% 0px', threshold: 0.01 });

  sections.forEach((section) => navObserver.observe(section));

  window.setInterval(() => {
    activeTestimonial.value = (activeTestimonial.value + 1) % testimonials.length;
  }, 4300);

  window.setInterval(() => {
    nextGallery();
  }, 3600);

  window.setInterval(() => {
    nextProcedure();
  }, 5800);
});
</script>

<template>
  <main>
    <transition name="loader-fade">
      <div v-if="isLoading" class="preloader" aria-label="Cargando sitio">
        <span class="brand-logo preloader-logo">
          <img :src="logoSrc" alt="Logo Dr. Gustavo Garrido" />
        </span>
      </div>
    </transition>

    <header class="site-header">
      <a class="brand" href="#inicio" aria-label="Inicio Dr. Gustavo Garrido">
        <span class="brand-logo"><img :src="logoSrc" alt="Logo Dr. Gustavo Garrido" /></span>
        <span>
          <strong>Dr. Gustavo Garrido</strong>
          <small>Vascular y Endovascular</small>
        </span>
      </a>
      <nav aria-label="Navegacion principal">
        <a href="#inicio" :class="{ active: activeSection === 'inicio' }">Inicio</a>
        <a href="#ayuda" :class="{ active: activeSection === 'ayuda' }">Enfermedades</a>
        <a href="#sobre-mi" :class="{ active: activeSection === 'sobre-mi' }">Sobre mi</a>
        <a href="#educacion" :class="{ active: activeSection === 'educacion' }">Educacion</a>
        <a href="#calculadora" :class="{ active: activeSection === 'calculadora' }">Riesgo vascular</a>
        <a href="#videos" :class="{ active: activeSection === 'videos' }">Galería</a>
        <a href="#ubicacion" :class="{ active: activeSection === 'ubicacion' }">Ubicacion</a>
        <a href="#agenda" :class="{ active: activeSection === 'agenda' }">Agendar</a>
      </nav>
    </header>

    <section
      id="inicio"
      class="hero"
      :style="{ '--hero-photo': `url(${heroDoctorImage})`, '--hero-vascular': `url(${heroImage})` }"
    >
      <div class="hero-content reveal is-visible">
        <p class="eyebrow">Cirugia Vascular y Endovascular</p>
        <h1 class="typewriter-title">
          {{ typedHeroTitle }}
          <span v-if="isTypingHeroTitle" class="typewriter-caret" aria-hidden="true"></span>
        </h1>
        <p class="hero-subtitle">
          Tratamiento de enfermedades arteriales, venosas y pie diabetico con diagnostico preciso, explicaciones sencillas y seguimiento personalizado.
        </p>
        <div class="hero-actions">
          <a class="btn primary" href="#agenda">Agendar una consulta</a>
          <a class="btn ghost" :href="whatsappUrl" target="_blank" rel="noopener">Escribirme por WhatsApp</a>
        </div>
      </div>
      <aside class="hero-panel reveal is-visible">
        <span>Agenda inmediata</span>
        <strong>{{ contact.phone }}</strong>
        <small>Valoracion vascular, segunda opinion y seguimiento por WhatsApp.</small>
      </aside>
    </section>

    <section class="trust-strip" aria-label="Areas de atencion">
      <span v-for="area in quickAreas" :key="area">{{ area }}</span>
    </section>

    <section id="ayuda" class="section reveal">
      <div class="section-heading">
        <p class="eyebrow dark">En que puedo ayudarte</p>
        <h2>Enfermedades complejas explicadas con lenguaje sencillo.</h2>
        <p>Cada condicion necesita una evaluacion distinta. Estas son las consultas vasculares mas frecuentes.</p>
      </div>
      <div class="condition-grid">
        <article v-for="condition in conditions" :key="condition.title" class="condition-card">
          <div class="condition-media" :class="condition.imageClass" aria-hidden="true">
            <span>{{ condition.title.charAt(0) }}</span>
          </div>
          <div class="condition-body">
            <span>{{ condition.title }}</span>
            <p>{{ condition.summary }}</p>
            <dl>
              <dt>Cuando preocuparse</dt>
              <dd>{{ condition.symptoms }}</dd>
              <dt>Como se maneja</dt>
              <dd>{{ condition.care }}</dd>
            </dl>
            <a :href="whatsappUrl" target="_blank" rel="noopener">Consultar este caso</a>
          </div>
        </article>
      </div>
    </section>

    <section id="sobre-mi" class="doctor-feature reveal">
      <div class="doctor-photo-card">
        <img
          v-if="doctorPhotoAvailable"
          :src="doctorPhotoSrc"
          alt="Dr. Gustavo Garrido atendiendo pacientes de cirugia vascular"
          @error="doctorPhotoAvailable = false"
        />
        <div v-else class="doctor-photo-placeholder">
          <span class="brand-logo"><img :src="logoSrc" alt="" /></span>
          <strong>Dr. Gustavo Garrido</strong>
        </div>
      </div>
      <div class="doctor-feature-copy">
        <p class="eyebrow dark">Quien soy</p>
        <h2>Experiencia medica sin discursos exagerados: diagnostico, explicacion y plan.</h2>
        <p>
          Mi practica se centra en escuchar al paciente, identificar el origen real de sus sintomas y proponer tratamientos proporcionales a su caso.
        </p>
        <div class="credential-grid">
          <span v-for="item in credentials" :key="item">{{ item }}</span>
        </div>
      </div>
    </section>

    <section class="section procedures reveal">
      <div class="procedure-heading">
        <div class="section-heading compact">
          <p class="eyebrow dark">Procedimientos</p>
          <h2>Opciones diagnósticas y terapéuticas para enfermedad venosa y arterial.</h2>
        </div>
        <div class="procedure-controls" aria-label="Controles de procedimientos">
          <button type="button" aria-label="Procedimiento anterior" @click="previousProcedure">‹</button>
          <button type="button" aria-label="Procedimiento siguiente" @click="nextProcedure">›</button>
        </div>
      </div>

      <div ref="procedureCarousel" class="procedure-carousel" aria-label="Carrusel de procedimientos">
        <article
          v-for="(procedure, index) in procedures"
          :key="procedure.title"
          class="procedure-card"
          :class="{ active: activeProcedure === index }"
          @click="selectProcedure(index)"
        >
          <span>{{ procedure.tag }}</span>
          <strong>{{ procedure.title }}</strong>
          <p>{{ procedure.description }}</p>
          <a :href="whatsappUrl" target="_blank" rel="noopener">Consultar</a>
        </article>
      </div>

      <div class="procedure-dots" aria-label="Seleccionar procedimiento">
        <button
          v-for="(procedure, index) in procedures"
          :key="procedure.title"
          type="button"
          :class="{ active: activeProcedure === index }"
          :aria-label="procedure.title"
          @click="selectProcedure(index)"
        ></button>
      </div>
    </section>

    <section id="educacion" class="education-section reveal">
      <div>
        <p class="eyebrow">Centro de educacion para pacientes</p>
        <h2>Una biblioteca vascular para resolver dudas antes de que se vuelvan urgencias.</h2>
        <p>Articulos breves, preguntas frecuentes, guias descargables e infografias pueden convertir el sitio en una referencia de salud vascular para Manabi.</p>
      </div>
      <div class="article-list">
        <article v-for="(article, index) in articles" :key="article.title" :class="{ open: activeArticle === index }">
          <button
            type="button"
            :aria-expanded="activeArticle === index"
            @click="activeArticle = activeArticle === index ? -1 : index"
          >
            <span>Artículo</span>
            <strong>{{ article.title }}</strong>
            <small aria-hidden="true">{{ activeArticle === index ? '−' : '+' }}</small>
          </button>
          <p v-if="activeArticle === index">{{ article.description }}</p>
        </article>
      </div>
    </section>

    <section id="calculadora" class="risk-calculator-section reveal">
      <div class="risk-calculator-copy">
        <p class="eyebrow dark">Calculadora de riesgo vascular</p>
        <h2>Una orientacion rapida para decidir si conviene consultar.</h2>
        <p>
          Ingresa edad, factores de riesgo y sintomas. La herramienta entrega una guia inicial para orientar al paciente, sin reemplazar una valoracion medica.
        </p>
        <div class="risk-alert">
          <strong>Atencion inmediata</strong>
          <span>Dolor intenso en reposo, pie frio, cambio brusco de color, heridas infectadas o hinchazon repentina deben valorarse pronto.</span>
        </div>
      </div>

      <div class="risk-calculator-card" aria-live="polite">
        <div class="risk-form-grid single">
          <label>
            Edad
            <input v-model="vascularRisk.age" type="number" min="0" max="110" inputmode="numeric" placeholder="Ej. 58" />
          </label>
        </div>

        <div class="risk-group">
          <strong>Factores de riesgo</strong>
          <div class="risk-check-grid">
            <label class="risk-check">
              <input v-model="vascularRisk.diabetes" type="checkbox" />
              Diabetes
            </label>
            <label class="risk-check">
              <input v-model="vascularRisk.hypertension" type="checkbox" />
              Hipertension
            </label>
            <label class="risk-check">
              <input v-model="vascularRisk.smoker" type="checkbox" />
              Tabaquismo
            </label>
            <label class="risk-check">
              <input v-model="vascularRisk.cholesterol" type="checkbox" />
              Colesterol alto
            </label>
          </div>
        </div>

        <div class="risk-group">
          <strong>Sintomas actuales</strong>
          <div class="risk-check-grid symptoms">
            <label v-for="symptom in riskSymptoms" :key="symptom.value" class="risk-check">
              <input v-model="vascularRisk.symptoms" type="checkbox" :value="symptom.value" />
              {{ symptom.label }}
            </label>
          </div>
        </div>

        <div class="risk-result" :class="vascularRiskResult.className">
          <span class="risk-score">Puntaje {{ vascularRiskResult.score }}</span>
          <strong>{{ vascularRiskResult.level }}</strong>
          <p>{{ vascularRiskResult.message }}</p>
          <small v-if="calculatedAge">Edad ingresada: {{ calculatedAge }} anos.</small>
          <div class="risk-actions">
            <a class="btn primary" :href="whatsappUrl" target="_blank" rel="noopener">{{ vascularRiskResult.action }}</a>
            <a class="btn dark" href="#agenda">Ir a agenda</a>
          </div>
        </div>

        <p class="risk-disclaimer">
          Esta calculadora es educativa y no emite diagnosticos. Si el sintoma es repentino, severo o progresivo, busca valoracion medica urgente.
        </p>
      </div>
    </section>

    <section id="videos" class="clinical-media-section reveal">
      <div class="clinical-media-heading">
        <p class="eyebrow">Galeria, videos y testimonios</p>
        <h2>Contenido visual para explicar, educar y generar confianza.</h2>
        <p>
          Una galeria clinica puede mostrar Doppler, angiografias, infografias, videos cortos y testimonios con autorizacion, sin perder privacidad ni elegancia.
        </p>
      </div>

      <div class="clinical-media-layout">
        <div class="clinical-video-stack">
          <article class="featured-video-card">
            <div class="featured-video-copy">
              <span>Video destacado</span>
              <strong>{{ instagramReels[activeVideo].title }}</strong>
            </div>
            <div class="featured-video-body">
              <video
                :key="instagramReels[activeVideo].src"
                :src="instagramReels[activeVideo].src"
                :title="instagramReels[activeVideo].title"
                controls
                preload="metadata"
                playsinline
              ></video>
              <div class="video-topic-list" aria-label="Videos educativos disponibles">
                <button
                  v-for="(reel, index) in instagramReels"
                  :key="reel.src"
                  type="button"
                  :class="{ active: index === activeVideo }"
                  @click="activeVideo = index"
                >
                  <span>{{ index + 1 }}</span>
                  {{ reel.title }}
                </button>
              </div>
            </div>
          </article>

          <article class="testimonial-video-card">
            <span>{{ testimonials[activeTestimonial].meta }}</span>
            <strong>{{ testimonials[activeTestimonial].title }}</strong>
            <p>{{ testimonials[activeTestimonial].text }}</p>
            <div class="testimonial-dots" aria-label="Seleccionar testimonio">
              <button
                v-for="(_, index) in testimonials"
                :key="index"
                type="button"
                :class="{ active: index === activeTestimonial }"
                :aria-label="`Testimonio ${index + 1}`"
                @click="activeTestimonial = index"
              ></button>
            </div>
          </article>
        </div>

        <div class="clinical-left-stack">
          <article class="gallery-carousel" aria-label="Galeria clinica educativa">
            <div
              class="gallery-stage"
              :class="galleryItems[activeGallery].tone"
              :style="{ '--gallery-image': `url(${galleryItems[activeGallery].image})` }"
            ></div>
            <div class="gallery-controls">
              <button type="button" aria-label="Elemento anterior" @click="previousGallery">‹</button>
              <div class="gallery-dots" aria-label="Seleccionar elemento de galeria">
                <button
                  v-for="(item, index) in galleryItems"
                  :key="item.title"
                  type="button"
                  :class="{ active: index === activeGallery }"
                  :aria-label="item.title"
                  @click="selectGallery(index)"
                ></button>
              </div>
              <button type="button" aria-label="Elemento siguiente" @click="nextGallery">›</button>
            </div>
          </article>

          <article class="testimonial-video-card">
            <span>{{ testimonials[activeTestimonial].meta }}</span>
            <strong>{{ testimonials[activeTestimonial].title }}</strong>
            <p>{{ testimonials[activeTestimonial].text }}</p>
            <div class="testimonial-dots" aria-label="Seleccionar testimonio">
              <button
                v-for="(_, index) in testimonials"
                :key="index"
                type="button"
                :class="{ active: index === activeTestimonial }"
                :aria-label="`Testimonio ${index + 1}`"
                @click="activeTestimonial = index"
              ></button>
            </div>
          </article>
        </div>
      </div>
    </section>

    <section id="ubicacion" class="section location-section reveal">
      <div class="location-copy">
        <p class="eyebrow dark">Ubicacion</p>
        <h2>Consulta vascular en Portoviejo.</h2>
        <p>Incluye mapa interactivo, telefonos, WhatsApp, horarios y referencia para llegar al consultorio.</p>
        <div class="location-details">
          <strong>{{ contact.address }}</strong>
          <span>{{ contact.city }}</span>
          <a :href="`tel:${contact.phone}`">{{ contact.phone }}</a>
          <a :href="`tel:${contact.phoneAlt}`">{{ contact.phoneAlt }}</a>
          <a :href="contact.instagramUrl" target="_blank" rel="noopener">{{ contact.instagram }}</a>
        </div>
        <div class="location-actions">
          <a class="btn primary" :href="contact.mapUrl" target="_blank" rel="noopener">Abrir Google Maps</a>
          <a class="btn dark" :href="whatsappUrl" target="_blank" rel="noopener">WhatsApp</a>
        </div>
      </div>
      <div class="map-frame">
        <iframe
          :src="contact.mapEmbedUrl"
          title="Ubicacion del consultorio del Dr. Gustavo Garrido"
          loading="lazy"
          referrerpolicy="no-referrer-when-downgrade"
        ></iframe>
      </div>
    </section>

    <section id="agenda" class="appointment reveal">
      <div class="appointment-copy">
        <p class="eyebrow">Agenda y segunda opinion</p>
        <h2>Solicita una consulta desde cualquier parte de la pagina.</h2>
        <p>Completa el formulario y se abrira WhatsApp con tu solicitud lista para enviar. El equipo confirmara disponibilidad.</p>
        <address>
          <strong>{{ contact.address }}</strong>
          <span>{{ contact.city }}</span>
          <a :href="`tel:${contact.phone}`">{{ contact.phone }}</a>
          <a :href="`tel:${contact.phoneAlt}`">{{ contact.phoneAlt }}</a>
          <a :href="`mailto:${contact.email}`">{{ contact.email }}</a>
        </address>
      </div>

      <form class="appointment-form" @submit.prevent="submitAppointment">
        <label>
          Nombre completo
          <input v-model="form.name" type="text" name="name" autocomplete="name" required />
        </label>
        <label>
          Telefono
          <input
            v-model="form.phone"
            type="tel"
            name="phone"
            autocomplete="tel"
            inputmode="numeric"
            pattern="[0-9]*"
            required
            @input="form.phone = form.phone.replace(/\D/g, '')"
          />
        </label>
        <label>
          Motivo de consulta
          <select v-model="form.reason" name="reason">
            <option>Valoracion vascular</option>
            <option>Varices o aranas vasculares</option>
            <option>Pie diabetico o heridas</option>
            <option>Trombosis venosa profunda</option>
            <option>Aneurisma de aorta</option>
            <option>Enfermedad arterial periferica</option>
            <option>Ulceras venosas o linfedema</option>
            <option>Segunda opinion</option>
          </select>
        </label>
        <label>
          Fecha preferida
          <input v-model="form.date" type="date" name="date" :min="today" />
        </label>
        <label class="full">
          Mensaje adicional
          <textarea v-model="form.message" name="message" rows="4" placeholder="Describe sintomas, tiempo de evolucion o estudios previos"></textarea>
        </label>
        <button class="btn primary full" type="submit">Enviar solicitud por WhatsApp</button>
        <p v-if="sent" class="form-note">Solicitud preparada. Confirma el envio en WhatsApp para coordinar tu cita.</p>
      </form>
    </section>

    <section id="faq" class="section faq reveal">
      <p class="eyebrow dark">Preguntas frecuentes</p>
      <h2>Informacion util antes de agendar.</h2>
      <details v-for="item in faq" :key="item.q">
        <summary>{{ item.q }}</summary>
        <p>{{ item.a }}</p>
      </details>
    </section>

    <a class="float-whatsapp" :href="whatsappUrl" target="_blank" rel="noopener" aria-label="Agendar por WhatsApp">
      <svg viewBox="0 0 32 32" aria-hidden="true" focusable="false">
        <path d="M16.02 3.2A12.65 12.65 0 0 0 5.17 22.36L3.6 28.8l6.61-1.51A12.65 12.65 0 1 0 16.02 3.2Zm0 22.84c-2.02 0-3.9-.59-5.49-1.61l-.38-.24-3.91.89.93-3.8-.25-.4a10.22 10.22 0 1 1 9.1 5.16Zm5.78-7.65c-.32-.16-1.88-.93-2.17-1.04-.29-.11-.5-.16-.72.16-.21.32-.83 1.04-1.02 1.25-.19.21-.37.24-.69.08-.32-.16-1.34-.49-2.55-1.56-.94-.84-1.58-1.88-1.77-2.2-.19-.32-.02-.49.14-.65.15-.15.32-.37.48-.56.16-.19.21-.32.32-.53.11-.21.05-.4-.03-.56-.08-.16-.72-1.74-.98-2.38-.26-.62-.52-.54-.72-.55h-.61c-.21 0-.56.08-.85.4-.29.32-1.12 1.09-1.12 2.67 0 1.57 1.15 3.09 1.31 3.3.16.21 2.26 3.45 5.48 4.84.77.33 1.36.53 1.83.68.77.24 1.47.21 2.02.13.62-.09 1.88-.77 2.15-1.52.27-.75.27-1.39.19-1.52-.08-.13-.29-.21-.61-.37Z" />
      </svg>
    </a>
    <button v-if="activeSection !== 'inicio'" class="back-to-top" type="button" aria-label="Volver arriba" @click="scrollToTop">↑</button>

    <footer class="footer">
      <div class="footer-brand">
        <span class="brand-logo footer-logo"><img :src="logoSrc" alt="Logo Dr. Gustavo Garrido" /></span>
        <div>
          <strong>Dr. Gustavo Garrido</strong>
          <span>{{ contact.specialty }}</span>
        </div>
      </div>
      <a href="#agenda">Agendar cita</a>
    </footer>
  </main>
</template>
