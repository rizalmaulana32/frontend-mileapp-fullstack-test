<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
import { Button } from '@/components/ui/button'
import { Label } from '@/components/ui/label'
import { CheckSquare } from 'lucide-vue-next'

const router = useRouter()
const username = ref('')
const password = ref('')
const error = ref('')
const loading = ref(false)

const handleLogin = async () => {
  error.value = ''
  loading.value = true

  try {
    const response = await fetch('https://backend-mileapp-fullstack-test.vercel.app/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        username: username.value,
        password: password.value,
      }),
    })

    const data = await response.json()

    if (response.ok) {
      localStorage.setItem('token', data.token)
      router.push('/tasks')
    } else {
      error.value = data.message || 'Login failed'
    }
  } catch (err) {
    error.value = 'Failed to connect to server'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-screen flex items-center justify-center relative overflow-hidden bg-gradient-to-br from-slate-950 via-blue-950 to-slate-900">
    <!-- Animated background elements -->
    <div class="absolute inset-0 overflow-hidden">
      <div class="absolute top-20 left-20 w-72 h-72 bg-blue-500/10 rounded-full blur-3xl animate-pulse-slow"></div>
      <div class="absolute bottom-20 right-20 w-96 h-96 bg-purple-500/10 rounded-full blur-3xl animate-pulse-slow" style="animation-delay: 1s"></div>
      <div class="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 w-64 h-64 bg-cyan-500/5 rounded-full blur-3xl animate-pulse-slow" style="animation-delay: 2s"></div>
    </div>

    <!-- Grid overlay -->
    <div class="absolute inset-0 bg-[linear-gradient(rgba(59,130,246,0.03)_1px,transparent_1px),linear-gradient(90deg,rgba(59,130,246,0.03)_1px,transparent_1px)] bg-[size:50px_50px] [mask-image:radial-gradient(ellipse_80%_50%_at_50%_50%,#000_70%,transparent_100%)]"></div>

    <Card class="w-full max-w-md relative z-10 bg-slate-900/80 backdrop-blur-xl border-slate-700/50 shadow-2xl shadow-blue-500/20 animate-scale-in">
      <CardHeader class="space-y-1 pb-6">
        <div class="flex items-center justify-center mb-4">
          <div class="p-3 bg-gradient-to-br from-blue-600 to-cyan-600 rounded-lg">
            <CheckSquare class="w-8 h-8 text-white" />
          </div>
        </div>
        <CardTitle class="text-2xl font-bold text-center text-white">
          Task Manager
        </CardTitle>
        <p class="text-center text-slate-400 text-sm">
          Sign in to continue
        </p>
      </CardHeader>
      <CardContent class="animate-slide-up">
        <form @submit.prevent="handleLogin" class="space-y-5">
          <div class="space-y-2 group">
            <Label for="username" class="text-slate-300">
              Username
            </Label>
            <Input
              id="username"
              v-model="username"
              type="text"
              placeholder="Enter username"
              required
              class="bg-slate-800/50 border-slate-700 text-white placeholder:text-slate-500 focus:border-blue-500 focus:ring-blue-500/20 transition-all duration-300 hover:border-slate-600"
            />
          </div>

          <div class="space-y-2 group">
            <Label for="password" class="text-slate-300">
              Password
            </Label>
            <Input
              id="password"
              v-model="password"
              type="password"
              placeholder="Enter password"
              required
              class="bg-slate-800/50 border-slate-700 text-white placeholder:text-slate-500 focus:border-blue-500 focus:ring-blue-500/20 transition-all duration-300 hover:border-slate-600"
            />
          </div>

          <div v-if="error" class="text-red-400 text-sm bg-red-500/10 border border-red-500/20 rounded-lg p-3 animate-slide-down">
            {{ error }}
          </div>

          <Button
            type="submit"
            class="w-full bg-gradient-to-r from-blue-600 to-cyan-600 hover:from-blue-700 hover:to-cyan-700 text-white font-semibold py-6 rounded-lg shadow-lg transition-all duration-200 hover:shadow-xl disabled:opacity-50 disabled:cursor-not-allowed"
            :disabled="loading"
          >
            <span v-if="!loading">Sign In</span>
            <span v-else class="flex items-center justify-center gap-2">
              <div class="w-4 h-4 border-2 border-white/30 border-t-white rounded-full animate-spin"></div>
              Signing in...
            </span>
          </Button>

          <div class="text-center text-sm text-slate-500 pt-2">
            Demo: <span class="text-blue-400 font-mono">admin</span> / <span class="text-blue-400 font-mono">admin123</span>
          </div>
        </form>
      </CardContent>
    </Card>
  </div>
</template>
