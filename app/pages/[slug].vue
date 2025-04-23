<script setup lang="ts">
import type { SanityDocument } from "@sanity/client";
import imageUrlBuilder from "@sanity/image-url";
import type { SanityImageSource } from "@sanity/image-url/lib/types/types";
import Nav from '~/nav.vue';

const CONTENT_QUERY = groq`*[_type == "content" && slug.current == $slug][0]`;
const { params } = useRoute();

const { data: content } = await useSanityQuery<SanityDocument>(CONTENT_QUERY, params);
const { projectId, dataset } = useSanity().client.config();
const urlFor = (source: SanityImageSource) =>
  projectId && dataset
    ? imageUrlBuilder({ projectId, dataset }).image(source)
    : null;

</script>


<template>
    <Nav />

    <main v-if="content" class="container mx-auto min-h-screen max-w-3xl p-8 flex flex-col gap-4">
        <a href="/" class="hover:underline">&larr; Back to content list</a>

        <img v-if="content.image" :src="urlFor(content.image)?.width(550).height(310).url()" :alt="content?.title"
            class="aspect-video rounded-xl" width="550" height="310" />

        <h1 v-if="content.title" class="text-4xl font-bold mb-8">{{ content.title }}</h1>

        <div class="prose">
            <p v-if="content.publishedAt">
                Published: {{ new Date(content.publishedAt).toLocaleDateString() }}<br />
                Author: {{ content.author }}
            </p>
            <br />
            <p v-if="content.excerpt"><small>{{ content.excerpt }}</small></p>
            <br />
            <SanityContent v-if="content.body" :blocks="content.body" />
        </div>
    </main>
</template>