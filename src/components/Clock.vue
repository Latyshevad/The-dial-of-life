<script>

import ModalWindow from './ModalWindow.vue'

export default{
  name: 'AnalogClock',
  data() {
    return {
      maxYearLife: 80,
      currentTime: new Date(),
      clockSize: 300,
      hourHandLength: 50,
      minuteHandLength: 70,
      secondHandLength: 85,
      isCustomTime: false,
      selectedDateTime: '',
      customHour: null,
      customMinute: null,
      customSecond: null,
      customDate: '',
      isModalVisible: false,
      modalContent: `
        <p>Если сжать всю человеческую жизнь (80 лет) в 24 часа, то где сейчас находитесь вы?</p>
        <p>Эта мини-программа визуально покажет вам тот момент где вы сейчас находитесь. Автор надеется, что этот метод поможет кому-нибудь преодолеть свой кризис. 
          Поможет осознать, что каждый возраст по своему прекрасен и нужно наслаждаться каждым моментом своей жизни. Нам всем предстоит многое успеть сделать
          до полуночи. Я искрене надеюсь, что все мы станем "полуночниками" и встретим новый рассвет на этих часах!</p>
        
        <p style="margin-top: 20px;"><strong>Внимание:</strong> Программа была создана только с развлекательной целью, автор никого не пытается оскорбить или обидеть. Всем добра!</p>
      `
    }
  },
  components: {
    ModalWindow,
  },
  computed: {
    hours() {
      return this.customHour ?? this.currentTime.getHours()
    },
    minutes() {
      return this.customMinute ?? this.currentTime.getMinutes()
    },
    seconds() {
      return this.customSecond ?? this.currentTime.getSeconds()
    },
    hourAngle() {
      return (this.hours % 12) * 30 + this.minutes * 0.5
    },
    minuteAngle() {
      return this.minutes * 6 + this.seconds * 0.1
    },
    secondAngle() {
      return this.seconds * 6
    },
    formattedTime() {
      return this.currentTime.toLocaleTimeString('ru-RU')
    },
    formattedDate() {
      return this.currentTime.toLocaleDateString('ru-RU', {
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    }
  },
  watch: {
    selectedDateTime(newValue) {
      if (newValue) {
        this.setCustomDateTime(newValue)
      }
    }
  },
  methods: {
    openModal() {
      this.isModalVisible = true
    },
    handleModalClose() {
      this.isModalVisible = false
    },
    getAngle(degrees) {
      return (degrees - 90) * Math.PI / 180
    },
    setCustomDateTime(datetime) {
      if (datetime) {
        this.currentTime = new Date(datetime)
        this.isCustomTime = true
        this.updateDateTimeInput()
      }
    },
    
    applyCustomDate() {
      if (this.customDate) {
        const newDate = new Date(this.customDate)
        const currentTime = this.currentTime
        newDate.setHours(currentTime.getHours(), currentTime.getMinutes(), currentTime.getSeconds())
        this.currentTime = newDate
        this.isCustomTime = true
        this.updateDateTimeInput()
      } else {
        alert('Пожалуйста, выберите дату')
      }
    },
    updateDateTimeInput() {
      // Форматируем дату для input datetime-local
      const year = this.currentTime.getFullYear()
      const month = String(this.currentTime.getMonth() + 1).padStart(2, '0')
      const day = String(this.currentTime.getDate()).padStart(2, '0')
      const hours = String(this.currentTime.getHours()).padStart(2, '0')
      const minutes = String(this.currentTime.getMinutes()).padStart(2, '0')
      this.selectedDateTime = `${year}-${month}-${day}T${hours}:${minutes}`
      
      // Обновляем поле даты
      this.customDate = `${year}-${month}-${day}`
      this.calculateLifeTime()
    },
    calculateLifeTime(year) { //Переделать
      let fullYears = (-1970) + new Date(( new Date() - new Date(this.currentTime) )).getFullYear()
      let lifeTime = (fullYears / this.maxYearLife) * 100
      let lifeHour = (24 / 100) * lifeTime

      let hour = Math.trunc(lifeHour)
      let minute = (Number((lifeHour - Math.trunc(lifeHour).toFixed(1))) * 60).toFixed(0)

      const newDate = new Date(this.currentTime)
      newDate.setHours(hour, minute, '00')
      this.currentTime = newDate
      this.isCustomTime = true
    }
  }
}
</script>


<template>
  <div class="clock-app">
    <div class="clock-container">
      <div class="time">Циферблат жизни</div>
      <div class="date">
        Если представить всю нашу жизнь в формате одних суток? Где вы сейчас находитесь?
        <button class="icon-btn" @click="openModal" aria-label="Информация">
          <span class="circle-icon">i</span>
        </button><br />
      </div>

      <!-- Аналоговые часы -->
      <div class="analog-clock" :style="{ width: clockSize + 'px', height: clockSize + 'px' }">
        <svg :width="clockSize" :height="clockSize" viewBox="0 0 200 200">
          <!-- Циферблат -->
          <circle cx="100" cy="100" r="95" fill="white" stroke="#333" stroke-width="2"/>
          
          <!-- Метки часов -->
          <g v-for="hour in 12" :key="hour">
            <line
              :x1="100 + 85 * Math.cos(getAngle(hour * 30))"
              :y1="100 + 85 * Math.sin(getAngle(hour * 30))"
              :x2="100 + 95 * Math.cos(getAngle(hour * 30))"
              :y2="100 + 95 * Math.sin(getAngle(hour * 30))"
              stroke="#333"
              stroke-width="2"
            />
            <text
              :x="100 + 75 * Math.cos(getAngle(hour * 30))"
              :y="100 + 75 * Math.sin(getAngle(hour * 30))"
              text-anchor="middle"
              dominant-baseline="middle"
              font-size="12"
              fill="#333"
            >
              {{ hour }}
            </text>
          </g>

          <!-- Секундные метки -->
          <g v-for="minute in 60" :key="'minute-' + minute">
            <line
              v-if="minute % 5 !== 0"
              :x1="100 + 92 * Math.cos(getAngle(minute * 6))"
              :y1="100 + 92 * Math.sin(getAngle(minute * 6))"
              :x2="100 + 96 * Math.cos(getAngle(minute * 6))"
              :y2="100 + 96 * Math.sin(getAngle(minute * 6))"
              stroke="#999"
              stroke-width="1"
            />
          </g>

          <!-- Часовая стрелка -->
          <line
            :x1="100"
            :y1="100"
            :x2="100 + hourHandLength * Math.cos(getAngle(hourAngle))"
            :y2="100 + hourHandLength * Math.sin(getAngle(hourAngle))"
            stroke="#333"
            stroke-width="4"
            stroke-linecap="round"
          />

          <!-- Минутная стрелка -->
          <line
            :x1="100"
            :y1="100"
            :x2="100 + minuteHandLength * Math.cos(getAngle(minuteAngle))"
            :y2="100 + minuteHandLength * Math.sin(getAngle(minuteAngle))"
            stroke="#333"
            stroke-width="3"
            stroke-linecap="round"
          />

          <!-- Секундная стрелка -->
          <line
            :x1="100"
            :y1="100"
            :x2="100 + secondHandLength * Math.cos(getAngle(secondAngle))"
            :y2="100 + secondHandLength * Math.sin(getAngle(secondAngle))"
            stroke="red"
            stroke-width="1.5"
            stroke-linecap="round"
          />

          <!-- Центральная точка -->
          <circle cx="100" cy="100" r="3" fill="#333"/>
        </svg>
      </div>

      <!-- Цифровое время -->
      <div class="digital-time">
        <div class="date">Вы родились: {{ formattedDate }}</div>
        <div class="time">Сейчас на ваших часах жизни: {{ formattedTime }}</div>
      </div>

      <!-- Управление датой и временем -->
      <div class="datetime-controls">
        <div class="control-group">
          <label>Ваша дата рождения:</label>
          <div class="date-inputs">
            <input 
              type="date" 
              v-model="customDate"
              class="date-input"
            />
            <button @click="applyCustomDate" class="apply-btn">
              Применить
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- Модальное окно -->
  <ModalWindow
    v-model:visible="isModalVisible"
    title='О приложении'
    :content="modalContent"
    @close="handleModalClose"
  />
</template>

<style scoped>
.clock-app {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  padding: 20px;
}

.clock-container {
  background: white;
  border-radius: 20px;
  padding: 30px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
  text-align: center;
  max-width: 500px;
  width: 100%;
}

.analog-clock {
  margin: 0 auto;
  background: white;
  border-radius: 50%;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.digital-time {
  margin-top: 30px;
  text-align: center;
}

.time {
  font-size: 3em;
  font-weight: bold;
  color: #333;
  font-family: 'Courier New', monospace;
  letter-spacing: 2px;
}

.date {
  font-size: 1.2em;
  color: #666;
  margin-top: 10px;
}

.datetime-controls {
  margin-top: 30px;
  padding-top: 20px;
  border-top: 1px solid #e0e0e0;
}

.control-group {
  margin-bottom: 20px;
  text-align: left;
}

.control-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
  color: #555;
  font-size: 0.9em;
}

.datetime-input {
  width: 100%;
  padding: 10px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  font-size: 1em;
  transition: border-color 0.3s;
}

.datetime-input:focus {
  outline: none;
  border-color: #667eea;
}

.quick-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.quick-btn {
  background: #f0f0f0;
  color: #333;
  border: none;
  padding: 8px 15px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.9em;
}

.quick-btn:hover {
  background: #e0e0e0;
  transform: translateY(-1px);
}

.quick-btn.current {
  background: #667eea;
  color: white;
}

.quick-btn.reset {
  background: #ff6b6b;
  color: white;
}

.time-inputs, .date-inputs {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.time-input {
  width: 70px;
  padding: 8px;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  text-align: center;
  font-size: 1em;
}

.time-input:focus, .date-input:focus {
  outline: none;
  border-color: #667eea;
}

.date-input {
  flex: 1;
  padding: 8px;
  border: 2px solid #e0e0e0;
  border-radius: 6px;
  font-size: 1em;
}

.apply-btn {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  padding: 8px 20px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.apply-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.4);
}

.status {
  margin-top: 15px;
  padding: 10px;
  background: #fff3cd;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}

.status-badge {
  background: #ffc107;
  color: #333;
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.85em;
  font-weight: 600;
}

.reset-link {
  background: none;
  border: none;
  color: #667eea;
  cursor: pointer;
  text-decoration: underline;
  font-size: 0.85em;
}

.reset-link:hover {
  color: #764ba2;
}

/* Стили для иконки "i" в кружке */
.icon-btn {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  margin: 10px;
}

.circle-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  background: #667eea;
  color: white;
  border-radius: 50%;
  font-size: 1.2rem;
  font-weight: bold;
  font-style: italic;
  transition: all 0.2s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  cursor: pointer;
}

.circle-icon:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}


@media (max-width: 768px) {
  .clock-container {
    padding: 20px;
  }
  
  .time {
    font-size: 2em;
  }
  
  .date {
    font-size: 1em;
  }
  
  .time-inputs, .date-inputs {
    flex-direction: column;
  }
  
  .time-input, .date-input, .apply-btn {
    width: 100%;
  }
  
  .quick-buttons {
    flex-direction: column;
  }
  
  .quick-btn {
    width: 100%;
  }

  .info-btn, .icon-btn {
    display: block;
    margin: 10px auto;
  }
}
</style>