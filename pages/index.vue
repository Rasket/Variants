<script setup lang="ts">
import { ref } from 'vue'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Textarea } from '@/components/ui/textarea'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { toast } from 'vue-sonner'
import { Trash2 } from "lucide-vue-next";



const handleDelete = (cardId: number) => {
  // Логика удаления
  let _toDelte = cards.value?.find(x => x.id === cardId);
  if (_toDelte) {
    let deleteIndex = cards.value.indexOf(_toDelte);
    cards.value?.splice(deleteIndex, 1);
  }
};

// Чеклист
const checklistItems = ref<string[]>([])
const newChecklistItem = ref('')

// Карточки с фото
const cards = ref<Array<{
  id: number
  image: string | null
  notes: string
  checklist: string[]
}>>([])

// Добавление пункта в чеклист
const addChecklistItem = () => {
  if (newChecklistItem.value.trim()) {
    checklistItems.value.push(newChecklistItem.value.trim())
    newChecklistItem.value = ''
  }
}

// Удаление пункта из чеклиста
const removeChecklistItem = (index: number) => {
  checklistItems.value.splice(index, 1)
}

// Добавление новой карточки
const addCard = () => {
  if (checklistItems.value.length === 0) {
    toast({
      title: 'Ошибка',
      description: 'Добавьте хотя бы один пункт в чеклист',
      variant: 'destructive'
    })
    return
  }
  
  cards.value.push({
    id: Date.now(),
    image: null,
    notes: '',
    checklist: [...checklistItems.value]
  })
}

// Обработка вставки изображения
const handlePasteImage = async (cardId: number, event: ClipboardEvent) => {
  event.preventDefault()
  const items = event.clipboardData?.items
  
  if (!items) {
    toast({
      title: 'Ошибка',
      description: 'Не удалось получить данные из буфера обмена',
      variant: 'destructive'
    })
    return
  }

  for (let i = 0; i < items.length; i++) {
    const item = items[i]
    if (item.type.startsWith('image')) {
      const blob = item.getAsFile()
      if (blob) {
        try {
          const reader = new FileReader()
          reader.onload = (e) => {
            const imageData = e.target?.result as string
            const cardIndex = cards.value.findIndex(c => c.id === cardId)
            if (cardIndex !== -1) {
              cards.value[cardIndex].image = imageData
              toast({
                title: 'Успех',
                description: 'Изображение успешно вставлено'
              })
            }
          }
          reader.onerror = () => {
            toast({
              title: 'Ошибка',
              description: 'Не удалось прочитать изображение',
              variant: 'destructive'
            })
          }
          reader.readAsDataURL(blob)
        } catch (error) {
          toast({
            title: 'Ошибка',
            description: 'Ошибка при обработке изображения',
            variant: 'destructive'
          })
        }
      }
      return // Обрабатываем только первое найденное изображение
    }
  }
  
  toast({
    title: 'Внимание',
    description: 'В буфере обмена нет изображения',
    variant: 'destructive'
  })
}

// Скачивание изображения
const downloadImage = (imageData: string) => {
  if (!imageData) {
    toast({
      title: 'Ошибка',
      description: 'Нет изображения для скачивания',
      variant: 'destructive'
    })
    return
  }
  
  try {
    const link = document.createElement('a')
    link.href = imageData
    link.download = `checklist-image-${Date.now()}.png`
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
    
    toast({
      title: 'Успех',
      description: 'Изображение скачивается'
    })
  } catch (error) {
    toast({
      title: 'Ошибка',
      description: 'Не удалось скачать изображение',
      variant: 'destructive'
    })
  }
}
</script>

<template>
  <div class="min-h-screen bg-slate-50 p-4">
    <div class="container mx-auto max-w-6xl">
      <h1 class="text-3xl font-bold mb-6">Варианты</h1>
      
      <!-- Форма добавления чеклиста -->
      <Card class="mb-8">
        <CardHeader>
          <CardTitle>Создайте ваш вариант с чеклистом</CardTitle>
        </CardHeader>
        <CardContent>
          <div class="flex gap-2 mb-4">
            <Input 
              v-model="newChecklistItem" 
              placeholder="Новый пункт чеклиста" 
              @keyup.enter="addChecklistItem" 
            />
            <Button @click="addChecklistItem">
              Добавить
            </Button>
          </div>
          
          <div v-if="checklistItems.length > 0" class="space-y-2 mb-4">
            <div 
              v-for="(item, index) in checklistItems" 
              :key="index" 
              class="flex items-center gap-2"
            >
              <input type="checkbox" class="h-4 w-4 rounded border-slate-300 text-slate-600 focus:ring-slate-600" />
              <span class="text-slate-800">{{ item }}</span>
              <Button 
                variant="ghost" 
                size="sm" 
                class="text-slate-500 hover:text-slate-900"
                @click="removeChecklistItem(index)"
              >
                <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                  <path d="M18 6 6 18" />
                  <path d="m6 6 12 12" />
                </svg>
              </Button>
            </div>
            
            <Button 
              class="mt-4 bg-slate-800 hover:bg-slate-900" 
              @click="addCard"
            >
              Создать карточку с текущим чеклистом
            </Button>
          </div>
          <p v-else class="text-slate-500">Добавьте пункты в чеклист</p>
        </CardContent>
      </Card>
      
      <!-- Список карточек -->
      <div class="space-y-6">
        <div 
          v-for="card in cards" 
          :key="card.id" 
          class="grid grid-cols-1 md:grid-cols-2 gap-4"
        >
          <Card>
            <CardHeader>
              <CardTitle>Фотография</CardTitle>
            </CardHeader>
            <CardContent>
              <div 
                @paste="handlePasteImage(card.id, $event)"
                @click="(e: MouseEvent) => (e.currentTarget as HTMLElement).focus()"
                tabindex="0"
                class="border-2 border-dashed border-slate-200 rounded-lg p-4 min-h-48 flex items-center justify-center cursor-pointer focus:border-slate-400 focus:outline-none focus:ring-2 focus:ring-slate-400 focus:ring-opacity-50 transition-colors"
                :class="{ 'bg-slate-100': !card.image }"
              >
                <div v-if="card.image" class="relative group w-full h-full flex items-center justify-center">
                  <img 
                    :src="card.image" 
                    alt="Вставленное изображение" 
                    class="max-h-64 max-w-full object-contain"
                  />
                  <Button 
                    variant="outline" 
                    class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity bg-white/90 hover:bg-white"
                    @click.stop="downloadImage(card.image)"
                  >
                    Скачать
                  </Button>
                </div>
                <p v-else class="text-slate-500 text-center">
                  Кликните сюда и вставьте изображение (Ctrl+V)<br>
                  <span class="text-sm">или перетащите файл</span>
                </p>
              </div>
            </CardContent>
          </Card>
          
          <Card>
            <CardHeader>
                <div class="flex items-center justify-between pb-2">
                <CardHeader class="p-0 whitespace-nowrap">
                  <CardTitle>Чеклист и заметки</CardTitle>
                </CardHeader>
                <Button
                  variant="ghost"
                  size="sm"
                  class="text-red-600 hover:text-red-800 hover:bg-red-50 cursor-pointer"
                  @click="handleDelete(card.id)"
                >
                  <span class="mr-2">Удалить</span>
                  <Trash2 class="h-4 w-4" />
                </Button>
              </div>
            </CardHeader>
            <CardContent class="space-y-4">
              <div v-if="card.checklist.length > 0" class="space-y-2">
                <div 
                  v-for="(item, index) in card.checklist" 
                  :key="index" 
                  class="flex items-center gap-2"
                >
                  <input 
                    type="checkbox" 
                    class="h-4 w-4 rounded border-slate-300 text-slate-600 focus:ring-slate-600"
                  />
                  <span class="text-slate-800">{{ item }}</span>
                </div>
              </div>
              <p v-else class="text-slate-500">Нет пунктов чеклиста</p>
              
              <div>
                <label class="block text-sm font-medium mb-1 text-slate-700">Заметки</label>
                <Textarea 
                  v-model="card.notes" 
                  placeholder="Добавьте заметки..." 
                  class="min-h-32 border-slate-300 focus:border-slate-400 focus:ring-slate-400"
                />
              </div>
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  </div>
</template>