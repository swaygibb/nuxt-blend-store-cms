<script setup lang="ts">
import type { SanityDocument } from "@sanity/client";
import imageUrlBuilder from "@sanity/image-url";
import type { SanityImageSource } from "@sanity/image-url/lib/types/types";
import Nav from '~/nav.vue';

const CONTENT_QUERY = groq`*[ 
  _type == "content" && defined(slug.current) 
] | order(publishedAt desc)[0...5] {
  _id, title, slug, publishedAt, image, author
}`;

const { data: content } = await useSanityQuery<SanityDocument[]>(CONTENT_QUERY);
const { projectId, dataset } = useSanity().client.config();

const urlFor = (source: SanityImageSource) =>
  projectId && dataset
    ? imageUrlBuilder({ projectId, dataset }).image(source)
    : null;
</script>

<template>
  <Nav />

  <main class="container mx-auto min-h-screen max-w-4xl p-8">
    <h1 class="text-4xl font-bold mb-8">News</h1>
    <div class="grid gap-6 sm:grid-cols-2 lg:grid-cols-3">
      <div
        v-for="item in content"
        :key="item._id"
        class="bg-white rounded-2xl shadow hover:shadow-md transition overflow-hidden"
      >
        <nuxt-link :to="`/${item.slug.current}`">
          <img
            v-if="item.image"
            :src="urlFor(item.image)?.width(550).height(310).url()"
            alt="Cover image"
            class="w-full h-48 object-cover"
          />
          <div class="p-4">
            <h2 class="text-xl font-semibold text-gray-500">{{ item.title }}</h2>
            <p class="text-sm text-gray-500">
              By {{ item.author }} <br />
              {{ new Date(item.publishedAt).toLocaleDateString() }}
            </p>
          </div>
        </nuxt-link>
      </div>
    </div>
  </main>
</template>
