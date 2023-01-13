<template>
    <div class="h-screen flex justify-center items-center flex-col">
        <h1 class="text-2xl">hello, world!</h1>
        <div class="flex mt-16  gap-5">
            <NuxtLink to="/detectImg" class="text-md border px-4 py-1 shadow-lg bg-slate-200 border-dotted">With img
            </NuxtLink>
            <NuxtLink to="/detectVideo" class="text-md border px-4 py-1 shadow-lg bg-slate-200 border-dotted">With Video
            </NuxtLink>
        </div>
    </div>
</template>