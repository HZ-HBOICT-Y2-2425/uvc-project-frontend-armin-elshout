<script lang="ts">
  import "/src/app.css";
  import { onMount } from 'svelte';
  import { page } from '$app/stores';

  let activity = { title: '', date: '', description: '', link: '' };
  let activityId: string | null = null;

  // Reviews State
  let reviews = [];
  let reviewer = '';
  let rating = 0;
  let comment = '';

  $: activityId = $page.url.pathname.split('/').pop();

  // Fetch activity and reviews on mount
  onMount(async () => {
    if (activityId) {
      await fetchActivity(activityId);
      await fetchReviews(activityId);
    }
  });

  async function fetchActivity(id: string) {
    try {
      const response = await fetch(`http://localhost:3011/api/activities/${id}`);
      if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);
      const data = await response.json();
      activity = {
        title: data.title || 'No title',
        date: data.date || 'No date',
        description: data.description || 'No description',
        link: data.link || ''
      };
    } catch (error) {
      console.error('Error fetching activity:', error);
    }
  }

  async function fetchReviews(id: string) {
    try {
      const response = await fetch(`http://localhost:3011/api/activities/${id}/reviews`);
      if (!response.ok) throw new Error(`HTTP error! Status: ${response.status}`);
      reviews = await response.json();
    } catch (error) {
      console.error('Error fetching reviews:', error);
    }
  }

  async function addReview() {
    if (!reviewer.trim() || !rating || !comment.trim()) {
      alert('All fields are required!');
      return;
    }

    const newReview = { activityId, reviewer, rating: Number(rating), comment };

    try {
      const response = await fetch('http://localhost:3011/api/reviews', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newReview),
      });

      if (!response.ok) throw new Error('Failed to submit review');

      const savedReview = await response.json();
      reviews = [...reviews, savedReview];

      // Clear form
      reviewer = '';
      rating = 0;
      comment = '';
    } catch (error) {
      console.error('Error submitting review:', error);
      alert('Failed to submit review.');
    }
  }
</script>


<main class="bg-green-500 p-4 sm:p-6 min-h-screen flex justify-center items-center">
  <div class="bg-white p-6 sm:p-8 rounded-lg shadow-lg max-w-3xl w-full">
    <!-- Header Section -->
    <h1 class="text-3xl sm:text-4xl font-extrabold text-gray-800 mb-6 text-center border-b pb-4">
      {activity.title}
    </h1>

    <!-- Activity Details Section -->
    <div class="space-y-4 text-gray-700 text-lg">
      <div>
        <span class="font-semibold text-gray-900">📅 Date:</span>
        <span class="ml-2 text-green-600">{activity.date}</span>
      </div>

      <div>
        <span class="font-semibold text-gray-900">📝 Description:</span>
        <p class="mt-2 text-gray-600 leading-relaxed">{activity.description}</p>
      </div>

      {#if activity.link}
        <div>
          <span class="font-semibold text-gray-900">🔗 More Info:</span>
          <a 
            href="{activity.link}" 
            target="_blank" 
            class="text-blue-600 hover:underline ml-2 break-all"
          >
            {activity.link}
          </a>
        </div>
      {/if}
    </div>

    <!-- Reviews Section -->
<div class="mt-8">
  <h2 class="text-2xl font-bold text-gray-800 mb-4">Reviews</h2>

  <!-- Existing Reviews -->
  {#if reviews.length > 0}
    <ul class="space-y-4">
      {#each reviews as review}
        <li class="border rounded-lg p-4 bg-gray-50 shadow-sm">
          <p class="text-lg font-semibold">{review.reviewer}</p>
          <p class="text-sm text-gray-600">⭐ {review.rating} / 5</p>
          <p class="mt-2 text-gray-700">{review.comment}</p>
        </li>
      {/each}
    </ul>
  {:else}
    <p class="text-gray-500">No reviews yet. Be the first to review!</p>
  {/if}
</div>

<!-- Add Review Form -->
<div class="mt-8">
  <h3 class="text-xl font-bold text-gray-800 mb-4">Add a Review</h3>
  <form class="space-y-4">
    <input
      type="text"
      placeholder="Your name"
      bind:value={reviewer}
      class="w-full border p-2 rounded"
    />
    <input
      type="number"
      min="1"
      max="5"
      placeholder="Rating (1-5)"
      bind:value={rating}
      class="w-full border p-2 rounded"
    />
    <textarea
      placeholder="Your review"
      bind:value={comment}
      class="w-full border p-2 rounded"
    ></textarea>
    <button
      type="button"
      on:click={addReview}
      class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600"
    >
      Submit Review
    </button>
  </form>
</div>


    <!-- Footer Section -->
    <div class="mt-8 text-center">
      <a href="/" 
         class="inline-block bg-green-600 text-white px-5 py-2 rounded-md hover:bg-green-700 transition duration-200">
        Back to Home
      </a>
    </div>
  </div>
</main>
