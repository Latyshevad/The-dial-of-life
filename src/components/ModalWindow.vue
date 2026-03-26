<template>
    <teleport to="body">
      <transition name="modal">
        <div v-if="isVisible" class="modal-overlay" @click.self="closeModal">
          <div class="modal-container">
            <!-- Заголовок -->
            <div class="modal-header">
              <h3 class="modal-title">{{ title }}</h3>
              <button class="modal-close-btn" @click="closeModal" aria-label="Закрыть">
                ✕
              </button>
            </div>
  
            <!-- Содержимое с прокруткой -->
            <div class="modal-body">
              <div class="modal-content" v-html="formattedContent"></div>
            </div>
  
            <!-- Кнопка закрытия -->
            <div class="modal-footer">
              <button class="modal-action-btn" @click="closeModal">
                Закрыть
              </button>
            </div>
          </div>
        </div>
      </transition>
    </teleport>
</template>
  
<script>
  export default {
    name: 'ModalWindow',
    props: {
      visible: {
        type: Boolean,
        default: false
      },
      title: {
        type: String,
        default: 'Информация'
      },
      content: {
        type: String,
        default: ''
      }
    },
    emits: ['update:visible', 'close'],
    computed: {
      isVisible: {
        get() {
          return this.visible
        },
        set(value) {
          this.$emit('update:visible', value)
        }
      },
      formattedContent() {
        // Поддерживаем HTML теги в контенте
        return this.content
      }
    },
    methods: {
      closeModal() {
        this.isVisible = false
        this.$emit('close')
      },
      handleEscape(e) {
        if (e.key === 'Escape' && this.isVisible) {
          this.closeModal()
        }
      }
    },
    watch: {
      isVisible(newVal) {
        if (newVal) {
          // Блокируем прокрутку body при открытии модального окна
          document.body.style.overflow = 'hidden'
          document.addEventListener('keydown', this.handleEscape)
        } else {
          // Возвращаем прокрутку body при закрытии
          document.body.style.overflow = ''
          document.removeEventListener('keydown', this.handleEscape)
        }
      }
    },
    beforeUnmount() {
      document.body.style.overflow = ''
      document.removeEventListener('keydown', this.handleEscape)
    }
  }
  </script>
  
  <style scoped>
  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
    backdrop-filter: blur(4px);
  }
  
  .modal-container {
    background: white;
    border-radius: 12px;
    width: 90%;
    max-width: 600px;
    max-height: 85vh;
    display: flex;
    flex-direction: column;
    box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
    animation: modalSlideIn 0.3s ease;
  }
  
  .modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 20px 24px;
    border-bottom: 1px solid #e5e7eb;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    border-radius: 12px 12px 0 0;
  }
  
  .modal-title {
    margin: 0;
    font-size: 1.25rem;
    font-weight: 600;
    color: white;
  }
  
  .modal-close-btn {
    background: rgba(255, 255, 255, 0.2);
    border: none;
    font-size: 1.5rem;
    cursor: pointer;
    color: white;
    width: 32px;
    height: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 8px;
    transition: all 0.2s;
  }
  
  .modal-close-btn:hover {
    background: rgba(255, 255, 255, 0.3);
    transform: scale(1.05);
  }
  
  .modal-body {
    flex: 1;
    overflow-y: auto;
    padding: 24px;
    max-height: calc(85vh - 130px);
  }
  
  .modal-content {
    line-height: 1.6;
    color: #374151;
    font-size: 0.95rem;
  }
  
  .modal-content ::deep h1,
  .modal-content ::deep h2,
  .modal-content ::deep h3 {
    margin-top: 0;
    margin-bottom: 12px;
    color: #1f2937;
  }
  
  .modal-content ::deep p {
    margin-bottom: 12px;
  }
  
  .modal-content ::deep ul,
  .modal-content ::deep ol {
    margin-bottom: 12px;
    padding-left: 24px;
  }
  
  .modal-content ::deep li {
    margin-bottom: 4px;
  }
  
  .modal-content ::deep code {
    background: #f3f4f6;
    padding: 2px 6px;
    border-radius: 4px;
    font-family: 'Courier New', monospace;
    font-size: 0.9em;
  }
  
  .modal-content ::deep pre {
    background: #f3f4f6;
    padding: 12px;
    border-radius: 6px;
    overflow-x: auto;
    margin-bottom: 12px;
  }
  
  .modal-content ::deep strong {
    font-weight: 600;
    color: #1f2937;
  }
  
  .modal-footer {
    padding: 16px 24px;
    border-top: 1px solid #e5e7eb;
    display: flex;
    justify-content: flex-end;
    background: #f9fafb;
    border-radius: 0 0 12px 12px;
  }
  
  .modal-action-btn {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    border: none;
    padding: 8px 20px;
    border-radius: 8px;
    font-size: 0.9rem;
    cursor: pointer;
    transition: all 0.2s;
  }
  
  .modal-action-btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
  }
  
  .modal-action-btn:active {
    transform: translateY(0);
  }
  
  /* Стили для скроллбара */
  .modal-body::-webkit-scrollbar {
    width: 8px;
  }
  
  .modal-body::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 4px;
  }
  
  .modal-body::-webkit-scrollbar-thumb {
    background: #c1c1c1;
    border-radius: 4px;
  }
  
  .modal-body::-webkit-scrollbar-thumb:hover {
    background: #a8a8a8;
  }
  
  /* Анимации */
  .modal-enter-active,
  .modal-leave-active {
    transition: opacity 0.3s ease;
  }
  
  .modal-enter-from,
  .modal-leave-to {
    opacity: 0;
  }
  
  .modal-enter-active .modal-container,
  .modal-leave-active .modal-container {
    transition: transform 0.3s ease;
  }
  
  .modal-enter-from .modal-container,
  .modal-leave-to .modal-container {
    transform: scale(0.95);
  }
  
  @keyframes modalSlideIn {
    from {
      transform: scale(0.95);
      opacity: 0;
    }
    to {
      transform: scale(1);
      opacity: 1;
    }
  }
  
  /* Адаптивность */
  @media (max-width: 640px) {
    .modal-container {
      width: 95%;
      max-height: 90vh;
    }
    
    .modal-header {
      padding: 16px 20px;
    }
    
    .modal-body {
      padding: 20px;
    }
    
    .modal-footer {
      padding: 12px 20px;
    }
    
    .modal-title {
      font-size: 1.1rem;
    }
  }
  </style>