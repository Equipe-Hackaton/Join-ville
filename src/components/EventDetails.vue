<template>
  <div class="event-details">
    <!-- Header com imagem -->
    <div class="event-header">
      <button @click="goBack" class="back-button">
        <font-awesome-icon icon="arrow-left" /> Voltar
      </button>

      <div class="event-banner">
        <img :src="event?.image" :alt="event?.title" />
        <div class="banner-overlay"></div>
      </div>
    </div>

    <!-- Container Principal -->
    <div class="event-container">
      <div class="event-layout">

        <!-- Coluna Esquerda: Informações -->
        <div class="event-main">
          <!-- Título e Categoria -->
          <div class="event-intro">
            <span class="event-badge">{{ event?.category }}</span>
            <h1 class="event-name">{{ event?.title }}</h1>

            <div class="event-meta">
              <div class="meta-item">
                <font-awesome-icon icon="calendar-days" class="meta-icon" />
                <span>{{ formatEventDate(event?.date) }}</span>
              </div>
              <div class="meta-item">
                <font-awesome-icon icon="clock" class="meta-icon" />
                <span>{{ event?.horario_inicio || '19:00' }}</span>
              </div>
              <div class="meta-item">
                <font-awesome-icon icon="location-dot" class="meta-icon" />
                <span>{{ event?.location }}</span>
              </div>
            </div>
          </div>

          <!-- Sobre o Evento -->
          <section class="event-section">
            <h2 class="section-heading">Sobre o evento</h2>
            <div class="section-content" v-html="formatDescription(event?.description)"></div>
          </section>

          <!-- Localização -->
          <section class="event-section">
            <h2 class="section-heading">
              <font-awesome-icon icon="location-dot" /> Localização
            </h2>
            <div class="section-content">
              <p class="location-name">{{ event?.location }}</p>
              <p class="location-address">Joinville - SC, Brasil</p>
              <div class="map-container">
                <font-awesome-icon icon="map-marked-alt" size="3x" />
                <p>Mapa da localização</p>
              </div>
            </div>
          </section>

          <!-- Organizador -->
          <section class="event-section">
            <h2 class="section-heading">
              <font-awesome-icon icon="building" /> Organizador
            </h2>
            <div class="organizer-box" @click="goToCompanyProfile">
              <div class="organizer-left">
                <div class="organizer-photo">
                  <img
                    v-if="event?.empresa?.avatar"
                    :src="event.empresa.avatar"
                    :alt="event.empresa.nome_empresa"
                  />
                  <font-awesome-icon v-else icon="building" />
                </div>
                <div class="organizer-details">
                  <h3 class="organizer-name">{{ event?.empresa?.nome_empresa || 'Empresa Organizadora' }}</h3>
                  <p class="organizer-bio">{{ event?.empresa?.descricao || 'Organizador de eventos em Joinville' }}</p>
                </div>
              </div>

              <div class="organizer-right" v-if="authStore.isAuthenticated && authStore.userType === 'USUARIO'">
                <button class="btn-follow" :class="{ following: isFollowing }" @click.stop="toggleFollow">
                  <font-awesome-icon :icon="isFollowing ? 'check' : 'plus'" />
                  {{ isFollowing ? 'Seguindo' : 'Seguir' }}
                </button>
                <button class="btn-message" @click.stop="startChat">
                  <font-awesome-icon icon="comment-dots" />
                  Enviar mensagem
                </button>
              </div>
            </div>
          </section>
        </div>

        <!-- Coluna Direita: Card Fixo -->
        <aside class="event-sidebar">
          <div class="sidebar-card">
            <!-- Preview da imagem -->
            <div class="card-image">
              <img :src="event?.image" :alt="event?.title" />
            </div>

            <!-- Informações -->
            <div class="card-body">
              <div class="card-info-grid">
                <div class="info-block">
                  <font-awesome-icon icon="calendar-days" class="info-icon" />
                  <div class="info-text">
                    <span class="info-label">Data</span>
                    <span class="info-value">{{ formatEventDate(event?.date) }}</span>
                  </div>
                </div>

                <div class="info-block">
                  <font-awesome-icon icon="clock" class="info-icon" />
                  <div class="info-text">
                    <span class="info-label">Horário</span>
                    <span class="info-value">{{ event?.horario_inicio || '19:00' }}</span>
                  </div>
                </div>

                <div class="info-block">
                  <font-awesome-icon icon="location-dot" class="info-icon" />
                  <div class="info-text">
                    <span class="info-label">Local</span>
                    <span class="info-value">{{ event?.location }}</span>
                  </div>
                </div>

                <div class="info-block">
                  <font-awesome-icon icon="tags" class="info-icon" />
                  <div class="info-text">
                    <span class="info-label">Categoria</span>
                    <span class="info-value">{{ event?.category }}</span>
                  </div>
                </div>
              </div>

              <!-- Status do Evento -->
              <div class="event-status" :class="event?.ativo ? 'active' : 'inactive'">
                <font-awesome-icon :icon="event?.ativo ? 'check-circle' : 'times-circle'" />
                {{ event?.ativo ? 'Evento Ativo' : 'Evento Encerrado' }}
              </div>

              <!-- Botão de Interesse -->
              <button
                v-if="authStore.userType !== 'EMPRESA'"
                class="btn-interest"
                @click="handleInterest"
              >
                <font-awesome-icon icon="heart" />
                Tenho Interesse
              </button>

              <!-- Ações -->
              <div class="card-actions">
                <button
                  class="action-btn"
                  :class="{ active: isFavorite(event?.id) }"
                  @click="toggleFavorite(event?.id)"
                >
                  <font-awesome-icon :icon="isFavorite(event?.id) ? 'heart' : ['far', 'heart']" />
                  Favoritar
                </button>
                <button class="action-btn" @click="shareEvent">
                  <font-awesome-icon icon="share-nodes" />
                  Compartilhar
                </button>
              </div>

              <!-- Nota -->
              <p class="card-note">
                <font-awesome-icon icon="info-circle" />
                Informações sobre ingressos serão enviadas pela empresa organizadora.
              </p>
            </div>
          </div>

          <!-- Social Share -->
          <div class="social-card">
            <h4>Compartilhar evento</h4>
            <div class="social-buttons">
              <button class="social-btn whatsapp" @click="shareWhatsApp" title="WhatsApp">
                <font-awesome-icon :icon="['fab', 'whatsapp']" />
              </button>
              <button class="social-btn facebook" @click="shareFacebook" title="Facebook">
                <font-awesome-icon :icon="['fab', 'facebook']" />
              </button>
              <button class="social-btn twitter" @click="shareTwitter" title="Twitter">
                <font-awesome-icon :icon="['fab', 'twitter']" />
              </button>
              <button class="social-btn link" @click="copyLink" title="Copiar link">
                <font-awesome-icon icon="link" />
              </button>
            </div>
          </div>
        </aside>
      </div>
    </div>

    <!-- Loading -->
    <div v-if="loading" class="loading-screen">
      <div class="spinner"></div>
      <p>Carregando evento...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import { useChatStore } from '@/stores/chat'
import { eventService } from '@/services/eventService'
import { useToast } from 'vue-toastification'
import api from '@/services/api'

const route = useRoute()
const router = useRouter()
const authStore = useAuthStore()
const chatStore = useChatStore()
const toast = useToast()

const event = ref(null)
const loading = ref(true)
const favoriteEvents = ref([])
const isFollowing = ref(false)

onMounted(async () => {
  await loadEvent()
  loadFavorites()
  checkIfFollowing()
})

const loadEvent = async () => {
  try {
    loading.value = true
    const allEvents = await eventService.getAllEvents()
    event.value = allEvents.find(e => e.id === parseInt(route.params.id))
    if (!event.value) {
      toast.error('Evento não encontrado')
      router.push('/')
    }
  } catch (error) {
    console.error('Erro ao carregar evento:', error)
    toast.error('Erro ao carregar evento')
  } finally {
    loading.value = false
  }
}

const loadFavorites = () => {
  const favorites = localStorage.getItem('favoriteEvents')
  if (favorites) favoriteEvents.value = JSON.parse(favorites)
}

const checkIfFollowing = () => {
  if (!authStore.isAuthenticated || !event.value?.empresa?.id) return
  const following = localStorage.getItem('followingCompanies')
  if (following) {
    const followingList = JSON.parse(following)
    isFollowing.value = followingList.includes(event.value.empresa.id)
  }
}

const isFavorite = (eventId) => favoriteEvents.value.includes(eventId)

const toggleFavorite = (eventId) => {
  const index = favoriteEvents.value.indexOf(eventId)
  if (index > -1) {
    favoriteEvents.value.splice(index, 1)
    toast.success('Removido dos favoritos')
  } else {
    favoriteEvents.value.push(eventId)
    toast.success('Adicionado aos favoritos')
  }
  localStorage.setItem('favoriteEvents', JSON.stringify(favoriteEvents.value))
}

const toggleFollow = () => {
  if (!authStore.isAuthenticated) {
    toast.warning('Faça login para seguir empresas')
    router.push('/login')
    return
  }
  const following = localStorage.getItem('followingCompanies')
  let followingList = following ? JSON.parse(following) : []
  const companyId = event.value.empresa.id
  const index = followingList.indexOf(companyId)
  if (index > -1) {
    followingList.splice(index, 1)
    isFollowing.value = false
    toast.success('Deixou de seguir')
  } else {
    followingList.push(companyId)
    isFollowing.value = true
    toast.success('Agora você está seguindo esta empresa')
  }
  localStorage.setItem('followingCompanies', JSON.stringify(followingList))
}

const startChat = async () => {
  if (!authStore.isAuthenticated) {
    toast.warning('Faça login para conversar')
    router.push('/login')
    return
  }
  if (authStore.userType !== 'USUARIO') {
    toast.error('Apenas usuários podem iniciar conversas')
    return
  }
  try {
    const empresaId = event.value.empresa?.id
    if (!empresaId) {
      toast.error('Empresa não encontrada')
      return
    }
    const room = await chatStore.startChatWith(empresaId)
    if (room && room.id) {
      router.push({ name: 'ChatRoom', params: { id: room.id } })
    }
  } catch (error) {
    console.error('Erro ao iniciar chat:', error)
    toast.error('Não foi possível iniciar o chat')
  }
}

const goToCompanyProfile = () => {
  if (!event.value?.empresa?.id) return
  router.push({ name: 'PublicCompanyProfile', params: { id: event.value.empresa.id } })
}

const handleInterest = async () => {
  if (!authStore.isAuthenticated) {
    toast.warning('Faça login para demonstrar interesse')
    router.push('/login')
    return
  }
  try {
    await api.showInterestInEvent(route.params.id)
    toast.success('Interesse registrado! A empresa pode entrar em contato.')
  } catch (error) {
    console.error('Erro ao registrar interesse:', error)
    const errorMsg = error.response?.data?.message || error.response?.data?.detail || 'Erro ao registrar interesse'
    toast.info(errorMsg)
  }
}

const shareEvent = () => {
  if (navigator.share) {
    navigator.share({
      title: event.value.title,
      text: `Confira: ${event.value.title}`,
      url: window.location.href
    })
  } else {
    copyLink()
  }
}

const shareWhatsApp = () => {
  const text = `Confira este evento: ${event.value.title} - ${window.location.href}`
  window.open(`https://wa.me/?text=${encodeURIComponent(text)}`, '_blank')
}

const shareFacebook = () => {
  window.open(`https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(window.location.href)}`, '_blank')
}

const shareTwitter = () => {
  const text = `Confira: ${event.value.title}`
  window.open(`https://twitter.com/intent/tweet?text=${encodeURIComponent(text)}&url=${encodeURIComponent(window.location.href)}`, '_blank')
}

const copyLink = () => {
  navigator.clipboard.writeText(window.location.href)
  toast.success('Link copiado!')
}

const formatEventDate = (dateStr) => {
  if (!dateStr) return 'Data a definir'
  if (dateStr.match(/\d{1,2}\s+[A-Z]{3}/)) {
    const meses = {
      'JAN': 'Janeiro', 'FEV': 'Fevereiro', 'MAR': 'Março',
      'ABR': 'Abril', 'MAI': 'Maio', 'JUN': 'Junho',
      'JUL': 'Julho', 'AGO': 'Agosto', 'SET': 'Setembro',
      'OUT': 'Outubro', 'NOV': 'Novembro', 'DEZ': 'Dezembro'
    }
    return dateStr.replace(/([A-Z]{3})/g, (match) => meses[match] || match)
  }
  return dateStr
}

const formatDescription = (desc) => {
  if (!desc) return 'Descrição não disponível'
  return desc.replace(/\n/g, '<br>')
}

const goBack = () => router.go(-1)
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.event-details {
  background: #f5f5f5;
  min-height: 100vh;
  padding-bottom: 4rem;
}

/* ========== HEADER ========== */
.event-header {
  position: relative;
  background: #fff;
  border-bottom: 1px solid #e0e0e0;
}

.back-button {
  position: absolute;
  top: 1rem;
  left: 1rem;
  z-index: 10;
  background: rgba(255,255,255,0.95);
  border: 1px solid #e0e0e0;
  padding: 0.75rem 1.5rem;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 600;
  color: #333;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: all 0.3s;
}

.back-button:hover {
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.event-banner {
  position: relative;
  width: 100%;
  height: 400px;
  overflow: hidden;
}

.event-banner img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.banner-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 150px;
  background: linear-gradient(to top, rgba(0,0,0,0.3), transparent);
}

/* ========== CONTAINER ========== */
.event-container {
  max-width: 1200px;
  margin: -80px auto 0;
  padding: 0 1.5rem;
  position: relative;
  z-index: 5;
}

.event-layout {
  display: grid;
  grid-template-columns: 1fr 380px;
  gap: 2rem;
  align-items: start;
}

/* ========== MAIN COLUMN ========== */
.event-main {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.event-intro {
  background: white;
  border-radius: 12px;
  padding: 2.5rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.event-badge {
  display: inline-block;
  background: #e3f2fd;
  color: #1976d2;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-size: 0.875rem;
  font-weight: 600;
  margin-bottom: 1rem;
}

.event-name {
  font-size: 2.25rem;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0 0 1.5rem 0;
  line-height: 1.2;
}

.event-meta {
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  color: #555;
  font-size: 1rem;
}

.meta-icon {
  color: #1976d2;
  font-size: 1.25rem;
}

/* ========== SECTIONS ========== */
.event-section {
  background: white;
  border-radius: 12px;
  padding: 2.5rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.section-heading {
  font-size: 1.5rem;
  font-weight: 700;
  color: #1a1a1a;
  margin: 0 0 1.5rem 0;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.section-content {
  color: #555;
  font-size: 1.05rem;
  line-height: 1.8;
}

.location-name {
  font-size: 1.25rem;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0 0 0.5rem 0;
}

.location-address {
  color: #777;
  margin: 0 0 1.5rem 0;
}

.map-container {
  background: #f5f5f5;
  border-radius: 8px;
  padding: 3rem;
  text-align: center;
  color: #999;
}

/* ========== ORGANIZER ========== */
.organizer-box {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 2rem;
  padding: 1.5rem;
  background: #f8f9fa;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s;
}

.organizer-box:hover {
  background: #e9ecef;
}

.organizer-left {
  display: flex;
  gap: 1.5rem;
  align-items: center;
  flex: 1;
}

.organizer-photo {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  overflow: hidden;
  background: white;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.organizer-photo img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.organizer-photo svg {
  font-size: 2rem;
  color: #1976d2;
}

.organizer-name {
  font-size: 1.125rem;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0 0 0.25rem 0;
}

.organizer-bio {
  font-size: 0.9375rem;
  color: #666;
  margin: 0;
  line-height: 1.4;
}

.organizer-right {
  display: flex;
  gap: 0.75rem;
}

.btn-follow,
.btn-message {
  padding: 0.65rem 1.25rem;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.875rem;
  cursor: pointer;
  transition: all 0.3s;
  white-space: nowrap;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.btn-follow {
  background: white;
  border: 2px solid #1976d2;
  color: #1976d2;
}

.btn-follow.following {
  background: #1976d2;
  color: white;
}

.btn-message {
  background: #28a745;
  border: 2px solid #28a745;
  color: white;
}

.btn-message:hover {
  background: #218838;
}

/* ========== SIDEBAR ========== */
.event-sidebar {
  position: sticky;
  top: 2rem;
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.sidebar-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.card-image {
  width: 100%;
  height: 220px;
  overflow: hidden;
}

.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.card-body {
  padding: 1.75rem;
}

.card-info-grid {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
  margin-bottom: 1.5rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid #e0e0e0;
}

.info-block {
  display: flex;
  align-items: start;
  gap: 1rem;
}

.info-icon {
  color: #1976d2;
  font-size: 1.25rem;
  margin-top: 0.125rem;
}

.info-text {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.info-label {
  font-size: 0.875rem;
  color: #777;
  font-weight: 500;
}

.info-value {
  font-size: 0.9375rem;
  color: #1a1a1a;
  font-weight: 600;
}

.event-status {
  padding: 0.75rem;
  border-radius: 8px;
  text-align: center;
  font-weight: 600;
  font-size: 0.9375rem;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.event-status.active {
  background: #d4edda;
  color: #155724;
}

.event-status.inactive {
  background: #f8d7da;
  color: #721c24;
}

.btn-interest {
  width: 100%;
  padding: 1rem;
  background: linear-gradient(135deg, #1976d2 0%, #1565c0 100%);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.0625rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.btn-interest:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(25,118,210,0.3);
}

.card-actions {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.75rem;
  margin-bottom: 1.5rem;
}

.action-btn {
  padding: 0.75rem;
  background: white;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.875rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: all 0.3s;
  color: #555;
}

.action-btn:hover {
  border-color: #1976d2;
  color: #1976d2;
}

.action-btn.active {
  background: #ff4757;
  border-color: #ff4757;
  color: white;
}

.card-note {
  font-size: 0.8125rem;
  color: #777;
  text-align: center;
  line-height: 1.4;
  margin: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}

.social-card {
  background: white;
  border-radius: 12px;
  padding: 1.75rem;
  box-shadow: 0 2px 12px rgba(0,0,0,0.08);
}

.social-card h4 {
  font-size: 1.0625rem;
  font-weight: 600;
  color: #1a1a1a;
  margin: 0 0 1rem 0;
}

.social-buttons {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 0.75rem;
}

.social-btn {
  aspect-ratio: 1;
  border: none;
  border-radius: 8px;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.3s;
}

.social-btn.whatsapp { background: #25D366; }
.social-btn.facebook { background: #1877F2; }
.social-btn.twitter { background: #1DA1F2; }
.social-btn.link { background: #6c757d; }

.social-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

/* ========== LOADING ========== */
.loading-screen {
  position: fixed;
  inset: 0;
  background: rgba(255,255,255,0.95);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 9999;
}

.spinner {
  width: 50px;
  height: 50px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #1976d2;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* ========== RESPONSIVE ========== */
@media (max-width: 1024px) {
  .event-layout {
    grid-template-columns: 1fr;
  }

  .event-sidebar {
    position: static;
  }
}

@media (max-width: 768px) {
  .event-banner {
    height: 300px;
  }

  .event-name {
    font-size: 1.75rem;
  }

  .event-intro,
  .event-section {
    padding: 1.75rem;
  }

  .organizer-box {
    flex-direction: column;
    align-items: start;
  }

  .organizer-right {
    width: 100%;
  }

  .btn-follow,
  .btn-message {
    flex: 1;
  }

  .card-actions {
    grid-template-columns: 1fr;
  }

  .event-meta {
    flex-direction: column;
    gap: 1rem;
  }
}

@media (max-width: 480px) {
  .event-container {
    padding: 0 1rem;
  }

  .event-name {
    font-size: 1.5rem;
  }

  .event-intro,
  .event-section {
    padding: 1.25rem;
  }

  .organizer-left {
    flex-direction: column;
    text-align: center;
  }

  .social-buttons {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
