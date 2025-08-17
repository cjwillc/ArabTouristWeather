import os
import requests
import pytz
from datetime import datetime
from telegram import Update, ParseMode
from telegram.ext import Updater, CommandHandler, CallbackContext, MessageHandler, Filters
from telegram.error import BadRequest
from apscheduler.schedulers.background import BackgroundScheduler
import logging

# Configuration
TELEGRAM_TOKEN = os.getenv('TELEGRAM_TOKEN', 'YOUR_BOT_TOKEN')
WEATHER_API_KEY = os.getenv('WEATHER_API_KEY', 'YOUR_WEATHER_API')
GROUP_CHAT_ID = os.getenv('GROUP_CHAT_ID', 'YOUR_GROUP_ID')  # Negative number for groups
ADMIN_IDS = [12345678]  # Your Telegram user ID

# Signature to add to all messages
AGENCY_SIGNATURE = "\n\n<i>تحيات فريق وكالة التنين للسفر و السياحة - منصة السياح العرب في فيتنام</i> 🐉"

# ... [keep all the existing CITIES, LANGUAGES, and other configurations] ...

class EnhancedWeatherBot:
    # ... [keep all existing methods] ...
    
    def generate_report(self, city, data, lang='ar'):
        temp = data['main']['temp']
        condition = data['weather'][0]['description']
        humidity = data['main']['humidity']
        tip = self.get_city_tip(city, data)
        
        report = f"""
<b>تقرير الطقس لـ {city}</b>
        
🌡 درجة الحرارة: {temp}°C
☁️ الحالة: {condition}
💧 الرطوبة: {humidity}%
        
💡 نصيحة اليوم:  
{tip}
"""
        return report + AGENCY_SIGNATURE  # Add signature here
    
    def start(self, update: Update, context: CallbackContext):
        lang = self.get_user_lang(update.effective_user.id)
        welcome_msg = LANGUAGES[lang]['welcome'] + AGENCY_SIGNATURE
        update.message.reply_text(welcome_msg, parse_mode=ParseMode.HTML)
    
    def weather_report(self, update: Update, context: CallbackContext):
        # ... [keep existing weather_report logic] ...
        update.message.reply_text(report + AGENCY_SIGNATURE, parse_mode=ParseMode.HTML)
        
    def broadcast(self, update: Update, context: CallbackContext):
        if not context.args:
            update.message.reply_text("Usage: /broadcast Your message" + AGENCY_SIGNATURE, parse_mode=ParseMode.HTML)
            return
            
        message = " ".join(context.args)
        self.updater.bot.send_message(
            chat_id=GROUP_CHAT_ID,
            text=f"📢 إعلان:\n\n{message}{AGENCY_SIGNATURE}",
            parse_mode=ParseMode.HTML
        )
    
    def handle_message(self, update: Update, context: CallbackContext):
        if "الطقس" in update.message.text:
            self.weather_report(update, context)
        else:
            update.message.reply_text(
                f"أهلاً بك! استخدم /weather لمعرفة حالة الطقس{AGENCY_SIGNATURE}",
                parse_mode=ParseMode.HTML
            )

    # ... [keep all other methods unchanged] ...

if __name__ == '__main__':
    bot = EnhancedWeatherBot()
    bot.run()
    
