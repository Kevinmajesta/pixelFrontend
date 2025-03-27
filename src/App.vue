<script setup>
import { ref, onMounted } from "vue";

const pixelSize = 3;
const gridWidth = 380;
const gridHeight = 170;
const selectedColor = ref("#000000");
const brushSize = ref(1);
let isDrawing = false;
let ctx = null;
let pixelsData = []; // Array untuk menyimpan warna setiap pixel

onMounted(() => {
  const canvas = document.getElementById("pixelCanvas");
  ctx = canvas.getContext("2d");
  clearCanvas();
});

const drawPixel = (x, y) => {
  if (!ctx) return;
  ctx.fillStyle = selectedColor.value;
  ctx.fillRect(
    x * pixelSize,
    y * pixelSize,
    pixelSize * brushSize.value,
    pixelSize * brushSize.value
  );

  // Simpan warna pixel di array
  pixelsData[y * gridWidth + x] = selectedColor.value;
};

const startDrawing = (event) => {
  isDrawing = true;
  draw(event);
};

const stopDrawing = () => {
  isDrawing = false;
};

const draw = (event) => {
  if (!isDrawing) return;
  const rect = event.target.getBoundingClientRect();
  const x = Math.floor((event.clientX - rect.left) / pixelSize);
  const y = Math.floor((event.clientY - rect.top) / pixelSize);
  drawPixel(x, y);
};

const clearCanvas = () => {
  if (!ctx) return;
  ctx.fillStyle = "#FFFFFF";
  ctx.fillRect(0, 0, gridWidth * pixelSize, gridHeight * pixelSize);
  pixelsData = new Array(gridWidth * gridHeight).fill("#FFFFFF"); // Reset pixel data
};

// Simpan ke Backend
const saveToBackend = async () => {
  const requestData = {
    pixels: pixelsData, // Array warna pixel
    gridWidth: gridWidth, // ✅ Kirim gridWidth
    gridHeight: gridHeight, // ✅ Kirim gridHeight
    pixelSize: pixelSize,
  };

  console.log(
    "Data yang dikirim ke backend:",
    JSON.stringify(requestData, null, 2)
  ); // ✅ Log sebelum mengirim request

  try {
    const response = await fetch("http://localhost:3000/save", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(requestData),
    });

    if (!response.ok) throw new Error("Gagal menyimpan pixel art!");

    // Buat blob agar bisa di-download
    const blob = await response.blob();
    const link = document.createElement("a");
    link.href = window.URL.createObjectURL(blob);
    link.download = "pixel-art.png";
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);

    console.log("Pixel Art berhasil disimpan dan diunduh!");
  } catch (error) {
    console.error("Error saat menyimpan Pixel Art:", error);
    alert("Gagal menyimpan Pixel Art.");
  }
};
</script>

<template>
  <div id="app">
    <h1 class="text-2xl font-bold mb-4 text-white-200">
      🎨 Pixel Art Editor (380x170)
    </h1>

    <!-- Warna Selector -->
    <div class="flex items-center gap-4 mb-4">
      <label class="text-gray-300 font-bold">Pilih Warna:</label>
      <input
        type="color"
        v-model="selectedColor"
        class="w-10 h-10 border border-gray-500 rounded-lg cursor-pointer"
      />
    </div>

    <!-- Pilihan Ukuran Brush -->
    <div class="flex items-center gap-4 mb-4 text-gray-300">
      <label class="font-bold">1 Pixel</label>
      <label v-for="size in [2, 3, 4, 5]" :key="size">
        <input type="radio" v-model="brushSize" :value="size" />
        {{ size }} Pixel
      </label>
    </div>

    <!-- Canvas -->
    <canvas
      id="pixelCanvas"
      :width="gridWidth * pixelSize"
      :height="gridHeight * pixelSize"
      class="border border-gray-300"
      @mousedown="startDrawing"
      @mouseup="stopDrawing"
      @mousemove="draw"
      @mouseleave="stopDrawing"
    ></canvas>

    <!-- Tombol Simpan & Hapus -->
    <div class="flex gap-4 mt-4">
      <button
        @click="saveToBackend"
        class="px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition"
      >
        Simpan Pixel Art 🎨
      </button>

      <button
        @click="clearCanvas"
        class="px-6 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600 transition"
      >
        Hapus Semua 🗑️
      </button>
    </div>
  </div>
</template>

<style></style>
