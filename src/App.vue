<script setup lang="ts">
import { ref, watchEffect } from 'vue'
import axios from 'axios'

const url = ref('https://ping.zenetys.com/api')
const messages = ref([])
const error = ref(null)
const interval = ref(1)
const cancel = ref(false)

const getlen = (str) => {
    const size = new Blob([str]).size;
    return size;
}

const getRequest = async (instance) => {
    if (instance === null) {
        const time = Date.now()
        const axiosInstance = createAxiosInstance(interval.value * 1000, url.value)
        console.log("instance created")
        cancel.value = false;
        messages.value = []
        return getRequest(axiosInstance)
    }

    if(cancel.value === true) {
        return
    }

    try {
        const res = await instance.get()
        const restr = JSON.stringify(res.data, null, 2)

        console.log(res)
        messages.value.push({
            data: restr,
            resize: getlen(restr),
            time: res.duration,
            status: res.status
        })
    } catch(err) {
        console.log(err)
        error.value = "There was an error, please verify the url"
        throw "There was an error" + err;

    }
    setTimeout(() => {
        getRequest(instance)
    }, instance.defaults.timeout)

}

const createAxiosInstance = (interval, url) => {
    const instance = axios.create({
        baseURL: url,
        timeout: interval,
        headers: {}
    });
    instance.interceptors.request.use(function (config) {

        config.metadata = { startTime: new Date()}
        return config;
    }, function (error) {
        return Promise.reject(error);
    });

    instance.interceptors.response.use(function (response) {

        response.config.metadata.endTime = new Date()
        response.duration = response.config.metadata.endTime - response.config.metadata.startTime
        return response;
    }, function (error) {
        error.config.metadata.endTime = new Date();
        error.duration = error.config.metadata.endTime - error.config.metadata.startTime;
        return Promise.reject(error);
    });
    return instance

}
</script>

<template class="bg-gray-700">
    <main class="w-11/12 xl:w-8/12 mx-auto mb-10 mt-10 bg-white flex flex-col">
        <input
        v-model="url"
        placeholder="Enter a url"
        class="rounded-lg border-2 w-full p-4 bg-gray-100 mb-2"
        >
        <input
        v-model="interval"
        placeholder="Choose an interval"
        class="rounded-lg border-2 w-full p-4 bg-gray-100"
        >
        <div class="my-4">
            <button class="px-4 py-1.5 bg-blue-500 text-lg rounded text-white mr-4" @click="getRequest(null)">Ping</button>
            <button class="px-4 py-1.5 bg-red-500 text-lg rounded text-white" @click="cancel = true">Cancel</button>

        </div>
        <p
        v-if="error"
        class="text-sm text-red-500"
        >{{error ? error : ''}}</p>

        <img src="./assets/loading.webp" v-if="cancel === false && messages.length > 0" alt="" class="w-1/6">
        <section class="flex flex-col-reverse">
            <div
            v-for="(message) in messages"
            class="px-4 py-1 bg-white-600 w-full  border border-green-600 my-1"
            >
            <div class="flex justify-between">
                <p>{{message.resize}} bytes received in {{message.time / 1000}} seconds</p>
                <button
                class="px-4 py-1.5 bg-green-500 text-lg rounded text-white my-4"
                @click="message.visible = !message.visible"
                >Show response</button>
            </div>
            <pre
            v-if="message.visible === true"
            class="my-5"
            >{{ message.data ? message.data : 'no message' }}</pre>
        </div>
    </section>

</main>
</template>
