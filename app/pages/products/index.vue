<script setup lang="ts">
import type { SanityDocument } from "@sanity/client";
import imageUrlBuilder from "@sanity/image-url";
import type { SanityImageSource } from "@sanity/image-url/lib/types/types";
import Nav from '~/nav.vue';

const PRODUCTS_QUERY = groq`*[ 
  _type == "product" && defined(slug.current) 
] | order(publishedAt desc)[0...5] {
  _id, name, slug, publishedAt, image, price, description
}`;

const { data: products } = await useSanityQuery<SanityDocument[]>(PRODUCTS_QUERY);
const { projectId, dataset } = useSanity().client.config();

const urlFor = (source: SanityImageSource) =>
  projectId && dataset
    ? imageUrlBuilder({ projectId, dataset }).image(source)
    : null;

const formatPrice = (price: number) =>
  new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(price);
</script>

<template>
  <Nav />

  <main class="container mx-auto min-h-screen max-w-4xl p-8">
    <h1 class="text-4xl font-bold mb-8">Products</h1>
    <div class="grid gap-6 sm:grid-cols-2 lg:grid-cols-3">
      <div
        v-for="item in products"
        :key="item._id"
        class="bg-white rounded-2xl shadow hover:shadow-md transition overflow-hidden"
      >
        <nuxt-link :to="`/products/${item.slug.current}`">
          <img
            v-if="item.image"
            :src="urlFor(item.image)?.width(550).height(310).url()"
            alt="Cover image"
            class="w-full h-48 object-cover"
          />
          <div class="p-4">
            <h2 class="text-xl font-semibold text-gray-500">{{ item.name }}</h2>
            <p class="text-sm text-gray-500">
              {{ formatPrice(item.price) }} <br />
              {{ new Date(item.publishedAt).toLocaleDateString() }}
            </p>
          </div>
        </nuxt-link>
      </div>
    </div>
  </main>
</template>
