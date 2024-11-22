<template>
    <v-app>
        <v-app-bar :elevation="2" image="https://i.pinimg.com/736x/9b/e0/bb/9be0bb33bc37ca9ffd71f23889f9f766.jpg" gradient="to bottom, rgba(0,0,0,0.5), rgba(0,0,0,0.9)"
                
                cover
                blur>
            <v-app-bar-title style="background-color: rgba(0, 0, 0, 0.6); color: white; text-align: center;display: flex; justify-content:center; font-size: 28px; font-family: as">THAMMANIT RINTHANG</v-app-bar-title>
        </v-app-bar>
        <div class="mt-12">
            <slot />
        </div>
    </v-app>
</template>