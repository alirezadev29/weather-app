<script setup>
defineProps({
  type: {
    type: String,
    required: true
  },
  size: {
    type: Number,
    default: 48
  }
});
</script>

<template>
  <svg
    :width="size"
    :height="size"
    viewBox="0 0 64 64"
    fill="none"
    xmlns="http://www.w3.org/2000/svg"
    class="outfit-icon"
  >
    <!-- تی‌شرت -->
    <g v-if="type === 'tshirt'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M22 12 L14 18 L18 26 L22 24 L22 52 L42 52 L42 24 L46 26 L50 18 L42 12 Q38 18 32 18 Q26 18 22 12 Z"/>
    </g>

    <!-- پیراهن آستین بلند -->
    <g v-else-if="type === 'shirt'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M22 12 L14 18 L18 26 L22 24 L22 52 L42 52 L42 24 L46 26 L50 18 L42 12 Q38 18 32 18 Q26 18 22 12 Z"/>
      <line x1="14" y1="18" x2="10" y2="46"/>
      <line x1="50" y1="18" x2="54" y2="46"/>
    </g>

    <!-- شلوار -->
    <g v-else-if="type === 'pants'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M20 10 L44 10 L44 54 L34 54 L32 32 L30 54 L20 54 Z"/>
      <line x1="20" y1="18" x2="44" y2="18" opacity="0.4"/>
    </g>

    <!-- شلوارک -->
    <g v-else-if="type === 'shorts'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M18 14 L46 14 L46 32 L36 32 L34 24 L30 24 L28 32 L18 32 Z"/>
    </g>

    <!-- کاپشن -->
    <g v-else-if="type === 'jacket'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M18 14 L26 10 Q32 14 38 10 L46 14 L50 24 L44 28 L44 54 L20 54 L20 28 L14 24 Z"/>
      <line x1="32" y1="14" x2="32" y2="54" stroke-dasharray="2 3" opacity="0.6"/>
    </g>

    <!-- پالتو -->
    <g v-else-if="type === 'coat'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M18 12 L26 8 Q32 12 38 8 L46 12 L50 24 L44 28 L44 56 L20 56 L20 28 L14 24 Z"/>
      <line x1="32" y1="12" x2="32" y2="56" stroke-dasharray="2 3" opacity="0.6"/>
      <circle cx="36" cy="32" r="1.5" fill="currentColor" stroke="none"/>
    </g>

    <!-- سویشرت -->
    <g v-else-if="type === 'sweater'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M18 14 L26 10 Q32 14 38 10 L46 14 L50 24 L44 28 L44 54 L20 54 L20 28 L14 24 Z"/>
      <path d="M26 10 Q32 16 38 10" opacity="0.5"/>
    </g>

    <!-- لباس زیر گرم -->
    <g v-else-if="type === 'thermal'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M18 14 L26 10 Q32 14 38 10 L46 14 L50 24 L44 28 L44 54 L20 54 L20 28 L14 24 Z"/>
      <line x1="22" y1="32" x2="42" y2="32" stroke-dasharray="2 3" opacity="0.6"/>
      <line x1="22" y1="40" x2="42" y2="40" stroke-dasharray="2 3" opacity="0.6"/>
      <line x1="22" y1="48" x2="42" y2="48" stroke-dasharray="2 3" opacity="0.6"/>
    </g>

    <!-- کفش -->
    <g v-else-if="type === 'shoes'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M14 40 L14 50 L50 50 L50 44 Q50 40 46 40 L36 40 L28 34 L20 34 Q14 34 14 40 Z"/>
    </g>

    <!-- کتانی -->
    <g v-else-if="type === 'sneakers'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M14 40 L14 50 L50 50 L50 44 Q50 40 46 40 L36 40 L28 34 L20 34 Q14 34 14 40 Z"/>
      <line x1="22" y1="40" x2="26" y2="40"/>
      <line x1="28" y1="40" x2="32" y2="40"/>
      <line x1="34" y1="42" x2="38" y2="42"/>
    </g>

    <!-- چکمه -->
    <g v-else-if="type === 'boots'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M20 10 L20 44 L14 50 L14 54 L50 54 L50 50 Q50 46 46 46 L34 46 L34 10 Z"/>
      <line x1="20" y1="20" x2="34" y2="20" opacity="0.4"/>
    </g>

    <!-- چتر -->
    <g v-else-if="type === 'umbrella'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M8 32 Q32 8 56 32 Q48 28 44 32 Q40 28 32 32 Q24 28 20 32 Q16 28 8 32 Z"/>
      <line x1="32" y1="32" x2="32" y2="52"/>
      <path d="M32 52 Q36 52 36 48" fill="none"/>
    </g>

    <!-- دستکش -->
    <g v-else-if="type === 'gloves'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M20 20 L20 44 Q20 48 24 48 L40 48 Q44 48 44 44 L44 20 Q44 16 40 16 L38 16 Q36 12 32 12 Q28 12 26 16 L24 16 Q20 16 20 20 Z"/>
      <line x1="26" y1="16" x2="26" y2="30" opacity="0.5"/>
      <line x1="32" y1="12" x2="32" y2="30" opacity="0.5"/>
      <line x1="38" y1="16" x2="38" y2="30" opacity="0.5"/>
    </g>

    <!-- کلاه -->
    <g v-else-if="type === 'hat'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M14 38 Q14 18 32 18 Q50 18 50 38 Z"/>
      <line x1="8" y1="38" x2="56" y2="38"/>
    </g>

    <!-- عینک آفتابی -->
    <g v-else-if="type === 'sunglasses'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <circle cx="20" cy="32" r="8"/>
      <circle cx="44" cy="32" r="8"/>
      <path d="M28 32 Q32 30 36 32" fill="none"/>
      <line x1="12" y1="28" x2="6" y2="26"/>
      <line x1="52" y1="28" x2="58" y2="26"/>
    </g>

    <!-- شال گردن -->
    <g v-else-if="type === 'scarf'" stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <path d="M14 22 Q14 18 18 18 L46 18 Q50 18 50 22 L50 28 L14 28 Z"/>
      <path d="M22 28 L22 52 L30 52 L30 28"/>
      <line x1="18" y1="23" x2="46" y2="23" opacity="0.4"/>
    </g>

    <!-- پیش‌فرض -->
    <g v-else stroke="currentColor" stroke-width="2" stroke-linejoin="round" stroke-linecap="round">
      <circle cx="32" cy="32" r="20"/>
      <circle cx="32" cy="32" r="8"/>
    </g>
  </svg>
</template>

<style scoped>
.outfit-icon {
  display: block;
  transition: transform 0.3s ease;
}

.outfit-icon:hover {
  transform: scale(1.05);
}
</style>