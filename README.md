const TelegramBot = require('node-telegram-bot-api');
 const token = '8555687054:AAEo8apGaIQz7WCn7aPkUzpL-_vGO1P7TxY';
 const bot = new TelegramBot(token, { polling: true });
 // ==================================================
 // ⚙️ CONFIGURACIÓN PRINCIPAL - EDITA AQUÍ TUS DATOS
 // ==================================================
 const CONFIG = {
   MENSAJE_BIENVENIDA: '👋 ¡Hola! Bienvenido a Passionline. ¿En qué puedo ayudarte hoy? Selecciona una opción del menú:',
   MENSAJE_PALABRA_PROHIBIDA: '⚠️ Lo siento, ese tipo de lenguaje no está permitido aquí. Por favor mantén la conversación respetuosa.',
   MENSAJE_DATOS_RECOLECTADOS: '✅ ¡Gracias por compartir tu información! Hemos guardado tus datos correctamente, pronto nos pondremos en contacto contigo.',
   
   // 🚫 PALABRAS PROHIBIDAS (agrega o quita las que necesites)
   PALABRAS_PROHIBIDAS: ['palabra1', 'palabra2', 'insulto', 'prohibido', 'ofensivo'],
   // 📝 RESPUESTAS AUTOMÁTICAS POR PALABRA CLAVE
   RESPUESTAS_AUTOMATICAS: {
     'hola': '👋 ¡Hola! ¿Cómo estás? Dime en qué te ayudo.',
     'precio': '💰 Nuestros precios varían según el servicio. ¿Te gustaría ver nuestra lista completa?',
     'ayuda': '🆘 Claro que sí, estoy aquí para ayudarte. Elige una opción del menú o escribe tu duda.',
     'contacto': '📞 Puedes contactarnos al número: +52 123 456 7890 o al correo: info@passionline.com',
     'gracias': '🤟 ¡Es un placer ayudarte! Si necesitas algo más, aquí estoy.'
   },
   // 📚 PUBLICACIONES POR TEMÁTICAS
   PUBLICACIONES: {
     'servicios': '🔹 Nuestros servicios incluyen:\n• Asesoría personalizada\n• Soporte 24/7\n• Contenido exclusivo\n• Acceso a comunidad privada',
     'informacion': 'ℹ️ Passionline es tu espacio dedicado a la pasión, conexión y contenido de calidad. Llevamos más de 5 años conectando personas.',
     'preguntas': '❓ Preguntas frecuentes:\n1. ¿Cómo me registro? -> Automático al escribirte\n2. ¿Tiene costo? -> Consulta nuestras tarifas\n3. ¿Es privado? -> 100% confidencial'
   }
 };
 // ==================================================
 // 📋 MENÚ PRINCIPAL Y SUBMENÚS
 // ==================================================
 const tecladoPrincipal = {
   reply_markup: {
     keyboard: [
       ['📌 Servicios', 'ℹ️ Información'],
       ['❓ Preguntas Frecuentes', '📞 Contacto'],
       ['📝 Dejar mis datos', '🔎 Ayuda']
     ],
     resize_keyboard: true,
     one_time_keyboard: false
   }
 };
 const tecladoSubMenu = {
   reply_markup: {
     keyboard: [
       ['🔙 Volver al Menú Principal', '📌 Ver detalles'],
       ['💬 Hablar con asesor']
     ],
     resize_keyboard: true
   }
 };
 // ==================================================
 // 🛡️ FILTRO DE PALABRAS PROHIBIDAS
 // ==================================================
 function contienePalabraProhibida(texto) {
   const textoMinusculas = texto.toLowerCase();
   return CONFIG.PALABRAS_PROHIBIDAS.some(palabra => textoMinusculas.includes(palabra.toLowerCase()));
 }
 // ==================================================
 // 📥 RECOLECCIÓN DE DATOS DE USUARIOS
 // ==================================================
 let estadoRecoleccion = {};
 function iniciarRecoleccion(chatId) {
   estadoRecoleccion[chatId] = { paso: 1, datos: {} };
   bot.sendMessage(chatId, '📝 Perfecto, vamos a registrar tus datos. Por favor dime tu *Nombre completo*:', { parse_mode: 'Markdown' });
 }
 function procesarRecoleccion(chatId, texto) {
   const estado = estadoRecoleccion[chatId];
   if (!estado) return false;
   switch (estado.paso) {
     case 1:
       estado.datos.nombre = texto;
       estado.paso = 2;
       bot.sendMessage(chatId, '✅ Nombre guardado. Ahora dime tu *Correo electrónico*:');
       break;
     case 2:
       estado.datos.correo = texto;
       estado.paso = 3;
       bot.sendMessage(chatId, '✅ Correo guardado. Finalmente, escribe tu *Teléfono de contacto*:');
       break;
     case 3:
       estado.datos.telefono = texto;
       // Aquí puedes guardar los datos en tu base o enviártelos
       console.log('📥 Datos recolectados:', estado.datos);
       bot.sendMessage(chatId, CONFIG.MENSAJE_DATOS_RECOLECTADOS, tecladoPrincipal);
       delete estadoRecoleccion[chatId];
       break;
   }
   return true;
 }
 // ==================================================
 // 🚀 FUNCIONAMIENTO PRINCIPAL DEL BOT
 // ==================================================
 // Mensaje de bienvenida al iniciar
 bot.onText(/\/start/, (msg) => {
   const chatId = msg.chat.id;
   bot.sendMessage(chatId, CONFIG.MENSAJE_BIENVENIDA, tecladoPrincipal);
 });
 // Escucha todos los mensajes
 bot.on('message', (msg) => {
   const chatId = msg.chat.id;
   const texto = msg.text || '';
   // Ignora comandos
   if (texto.startsWith('/')) return;
   // 🛡️ Verificar palabras prohibidas
   if (contienePalabraProhibida(texto)) {
     bot.sendMessage(chatId, CONFIG.MENSAJE_PALABRA_PROHIBIDA);
     return;
   }
   // 📥 Si está en proceso de recolección de datos
   if (procesarRecoleccion(chatId, texto)) {
     return;
   }
   // 📋 RESPUESTAS POR BOTONES DEL MENÚ
   switch (texto) {
     case '📌 Servicios':
       bot.sendMessage(chatId, CONFIG.PUBLICACIONES.servicios, tecladoSubMenu);
       break;
     case 'ℹ️ Información':
       bot.sendMessage(chatId, CONFIG.PUBLICACIONES.informacion, tecladoSubMenu);
       break;
     case '❓ Preguntas Frecuentes':
       bot.sendMessage(chatId, CONFIG.PUBLICACIONES.preguntas, tecladoSubMenu);
       break;
     case '📞 Contacto':
       bot.sendMessage(chatId, '📞 Puedes escribirnos a: info@passionline.com\n📱 Teléfono: +52 123 456 7890\n⏰ Atención: 8:00 AM - 10:00 PM', tecladoSubMenu);
       break;
     case '📝 Dejar mis datos':
       iniciarRecoleccion(chatId);
       break;
     case '🔎 Ayuda':
       bot.sendMessage(chatId, '🆘 ¿Necesitas ayuda? Escribe tu duda o elige una opción:\n• Servicios disponibles\n• Cómo funciona\n• Problemas técnicos', tecladoPrincipal);
       break;
     case '🔙 Volver al Menú Principal':
       bot.sendMessage(chatId, '👌 Volviendo al menú principal...', tecladoPrincipal);
       break;
     case '📌 Ver detalles':
       bot.sendMessage(chatId, '🔍 Aquí tienes más detalles sobre lo que consultaste. ¿Hay algo más que quieras saber?', tecladoSubMenu);
       break;
     case '💬 Hablar con asesor':
       bot.sendMessage(chatId, '👤 En un momento un asesor se conectará contigo. Por favor espera un instante...', tecladoPrincipal);
       break;
     default:
       // ✨ RESPUESTAS AUTOMÁTICAS POR PALABRA CLAVE
       let respuestaEncontrada = false;
       Object.keys(CONFIG.RESPUESTAS_AUTOMATICAS).forEach(palabra => {
         if (texto.toLowerCase().includes(palabra.toLowerCase())) {
           bot.sendMessage(chatId, CONFIG.RESPUESTAS_AUTOMATICAS[palabra]);
           respuestaEncontrada = true;
         }
       });
       // Si no coincide con nada
       if (!respuestaEncontrada) {
         bot.sendMessage(chatId, '🤔 Disculpa, no entendí bien lo que me dijiste. ¿Podrías explicarlo de otra forma o elegir una opción del menú?', tecladoPrincipal);
       }
       break;
   }
 });
 console.log('✅ Bot de Passionline activado y funcionando correctamente con el token proporcionado');
