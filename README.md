# SQL-va-PostgreSQL-Asoslari-2
Mana siz aytgan barcha talablarga javob beradigan, toza va zamonaviy Foydalanuvchi Profili Kartasi (Profile Card) uchun HTML va Tailwind CSS kodlari.

Kodni to'g'ridan-to'g'ri loyihangizga nusxalab olishingiz yoki Tailwind o'yin maydonchasida (Playground) sinab ko'rishingiz mumkin.

HTML & Tailwind CSS Kod
HTML
<div class="flex items-center justify-center min-h-screen bg-gray-100 p-4">
  
  <div class="max-w-sm w-full bg-white rounded-2xl shadow-xl p-6 text-center border border-gray-100">
    
    <div class="relative w-24 h-24 mx-auto mb-4">
      <img 
        src="https://images.unsplash.com/photo-1534528741775-53994a69daeb?q=80&w=256&auto=format&fit=crop" 
        alt="Profil rasmi" 
        class="w-full h-full object-cover rounded-full ring-4 ring-indigo-50 transition-transform duration-300 hover:scale-105 cursor-pointer"
      />
    </div>

    <h2 class="text-xl font-bold text-gray-850 tracking-tight transition-colors hover:text-indigo-600 cursor-pointer">
      Malika Axmedova
    </h2>
    <p class="text-sm font-semibold text-indigo-600 mt-1 uppercase tracking-wider">
      Lead UX/UI Designer
    </p>

    <p class="text-sm text-gray-500 mt-3 mb-6 leading-relaxed">
      Raqamli mahsulotlar uchun chiroyli va qulay interfeyslar yarataman. Minimalizm va foydalanuvchi tajribasiga e'tibor qarataman.
    </p>

    <div class="grid grid-cols-2 gap-3">
      
      <button class="bg-indigo-600 text-white text-sm font-medium py-2.5 px-4 rounded-xl shadow-sm hover:bg-indigo-700 active:bg-indigo-800 transition-all duration-200 transform hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
        Bog'lanish
      </button>
      
      <button class="bg-transparent border border-gray-300 text-gray-700 text-sm font-medium py-2.5 px-4 rounded-xl hover:bg-gray-50 hover:border-gray-400 active:bg-gray-100 transition-all duration-200 transform hover:-translate-y-0.5 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
        Portfolio
      </button>

    </div>

  </div>

</div>
Amamalga oshirilgan talablar tahlili:
Avatar (w-24 h-24 rounded-full): Tailwind'da w-24 va h-24 klasslari aniq 96px o'lchamni beradi. rounded-full esa uni butunlay yumaloq qiladi.

Ism va Lavozim: Ism uchun text-xl font-bold (katta va qalin), lavozim uchun esa ajralib turishi uchun text-sm font-semibold va boshqa rang ishlatildi.

Bio matni (text-gray-500): Matn ko'zni charchatmasligi uchun yumshoq kulrang rangga o'tkazildi.

Tugmalar: Biri to'q ko'k fondagi Primary (bg-indigo-600), ikkinchisi esa shaffof fondagi chegarali Outline (border border-gray-300) uslubida yaratildi.

Hover effektlar (hover:...): Rasm ustiga kelganda biroz kattalashadi (hover:scale-105), tugmalar rangi to'qishadi va biroz yuqoriga ko'tariladi (hover:-translate-y-0.5).

Karta asosi (bg-white shadow-xl rounded-2xl): Oq fon, kuchliroq soya (shadow) va chiroyli yumaloq burchaklar berildi.
