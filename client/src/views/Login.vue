<template>
  <div class="p-6 max-w-xl mx-auto">
    <h2 class="text-2xl font-bold text-black mb-4">Login</h2>
    <form @submit.prevent="login" class="space-y-4">
      <div>
        <label class="block text-black mb-1">Username</label>
        <input
          v-model="username"
          type="text"
          required
          placeholder="Username"
          class="w-full px-3 py-2 rounded border border-gray-300 text-black"
        />
      </div>
      <div>
        <label class="block text-black mb-1">Password</label>
        <input
          v-model="password"
          type="password"
          required
          placeholder="Password"
          class="w-full px-3 py-2 rounded border border-gray-300 text-black"
        />
      </div>
      <div class="flex justify-end space-x-2">
        <button
          type="submit"
          class="bg-white text-black px-4 py-2 rounded border border-gray-300 hover:bg-gray-200 transition"
        >
          Login
        </button>
      </div>
    </form>
    <p v-if="error" class="text-red-400 mt-4">{{ error }}</p>
  </div>
</template>

<script setup>
import { ref } from "vue";
import api from "@/api/axios";
import { useRoute, useRouter } from "vue-router";
import { useAuthStore } from "@/stores/authStore";
import { useWorkspaceStore } from "@/stores/workspaceStore";

const workspaceStore = useWorkspaceStore();
const username = ref("");
const password = ref("");
const error = ref("");

const route = useRoute();
const router = useRouter();
const auth = useAuthStore();

const login = async () => {
  try {
    const res = await api.post("/api/auth/login", {
      username: username.value,
      password: password.value,
    });

    auth.login(res.data.token, username.value); // Use actual username if returned
    var redirectPath = route.query.redirect || "/"; // fallback to homepage or dashboard
    if (redirectPath == "/") {
      // If redirecting to homepage, check if user has a workspace
      await workspaceStore.fetchWorkspaces();
      if (workspaceStore.currentWorkspaceId) {
        redirectPath = `/workspaces/${workspaceStore.currentWorkspaceId}`;
      } else {
        redirectPath = "/workspaces/create"; // Redirect to workspaces if no current workspace
      }
    }
    router.push(redirectPath);
  } catch (err) {
    error.value = "Login failed";
  }
};
</script>
