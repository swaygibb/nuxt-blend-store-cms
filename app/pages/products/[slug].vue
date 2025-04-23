<script setup lang="ts">
import type { SanityDocument } from "@sanity/client";
import imageUrlBuilder from "@sanity/image-url";
import type { SanityImageSource } from "@sanity/image-url/lib/types/types";
import Nav from '~/nav.vue';

const PRODUCT_QUERY = groq`*[_type == "product" && slug.current == $slug][0]`;
const { params } = useRoute();

const { data: product } = await useSanityQuery<SanityDocument>(PRODUCT_QUERY, params);
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

  <main v-if="product" class="container mx-auto min-h-screen max-w-3xl p-8 flex flex-col gap-4">
    <a href="/products" class="hover:underline">&larr; Back to product list</a>

    <img v-if="product.image" :src="urlFor(product.image)?.width(550).height(310).url()" :alt="product?.name"
      class="aspect-video rounded-xl" width="550" height="310" />

    <h1 v-if="product.name" class="text-4xl font-bold mb-8">{{ product.name }}</h1>

    <div class="prose">
      <p v-if="product.publishedAt">
        Published: {{ new Date(product.publishedAt).toLocaleDateString() }}<br />
        Price: {{ formatPrice(product.price) }}
      </p>
      <br />
      <p v-if="product.excerpt"><small>{{ product.excerpt }}</small></p>
      {{ product.description }}
    </div>
  </main>
</template>