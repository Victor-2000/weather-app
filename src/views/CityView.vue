<template>
        <div>
            <Transition name="cityview">
                <Suspense>
                    <AsyncCityView/>
                    <template #fallback>
                        <CityViewSkeleton/>
                    </template>
                </Suspense>
            </Transition>
        </div>
</template>

<script setup>
import AsyncCityView from '../components/AsyncCityView.vue';
import CityViewSkeleton from '../components/CityViewSkeleton.vue';
</script>

<style>
    .cityview-enter-active {
    transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02) 0.15s;
    }

    .cityview-leave-active {
    transition: all 0.3s cubic-bezier(0.52, 0.02, 0.19, 1.02);
    }

    .cityview-enter-from {
        opacity: 1;
    }

    .cityview-leave-to {
        opacity: 0;
    }
</style>