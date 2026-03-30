import streamlit as st
from openai import OpenAI
import base64
import json

# Инициализация клиента OpenRouter (он использует формат OpenAI)
# Ключ будет надежно браться из секретов Streamlit
client = OpenAI(
    base64_url="https://openrouter.ai/api/v1",
    api_key=st.secrets["OPENROUTER_API_KEY"],
)

def encode_image(image_file):
    """Конвертирует загруженную картинку в Base64 для передачи в API"""
    return base64.b64encode(image_file.read()).decode('utf-8')

# Настройка интерфейса
st.set_page_config(page_title="Instagram Alt-Tag Generator", page_icon="📸")
st.title("генератор Alt-тегов для Instagram (Powered by Qwen-VL)")

st.markdown("Загрузите картинку и получите идеальный SEO-оптимизированный alt-тег.")

# Выбор формата поста
post_format = st.radio(
    "Выберите формат вашего поста:",
    ("Одиночное фото (Single Photo)", "Карусель (Carousel)", "Обложка/кадр из Reels")
)

# Загрузка изображения
uploaded_file = st.file_uploader("Загрузите изображение (JPG, PNG)", type=['jpg', 'jpeg', 'png'])

if uploaded_file is not None:
    # Показываем превью картинки
    st.image(uploaded_file, caption="Ваше изображение", use_container_width=True)
    
    if st.button("Сгенерировать Alt-тег 🚀"):
        with st.spinner("Qwen анализирует изображение..."):
            try:
                # Конвертируем картинку
                base64_image = encode_image(uploaded_file)
                image_url = f"data:image/jpeg;base64,{base64_image}"
                
                # Системный промт из нашей задумки
                system_prompt = f"""Ты — SMM-специалист и SEO-маркетолог. Составь alt-тег для этого изображения.
Формат поста: {post_format}. 
Правила: описывай факты, объекты, действия, цвета и текст на фото. Без "На фото изображено". Максимум 100-120 слов. Добавь 3-5 ключевых слов.
Верни ответ СТРОГО в формате JSON: {{"alt_text": "твой текст", "keywords": ["ключ1", "ключ2"]}}"""

                # Запрос к бесплатной модели Qwen на OpenRouter
                response = client.chat.completions.create(
                    model="qwen/qwen-2.5-vl-7b-instruct:free",
                    messages=[
                        {
                            "role": "user",
                            "content": [
                                {"type": "text", "text": system_prompt},
                                {"type": "image_url", "image_url": {"url": image_url}}
                            ]
                        }
                    ],
                    response_format={"type": "json_object"}
                )
                
                # Обработка ответа
                result_text = response.choices[0].message.content
                result_json = json.loads(result_text)
                
                st.success("Готово!")
                st.subheader("Текст для Alt-тега:")
                st.write(result_json.get("alt_text", "Текст не сгенерирован"))
                
                st.subheader("SEO Ключевые слова:")
                st.write(", ".join(result_json.get("keywords", [])))
                
            except Exception as e:
                st.error(f"Произошла ошибка: {e}")
