<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let candidates = [];
  let totalVotes = 0;
  let loading = true;

  onMount(async () => {
    const { data } = await supabase
      .from('candidates')
      .select('*')
      .order('vote_count', { ascending: false });
    
    candidates = data || [];
    totalVotes = candidates.reduce((sum, c) => sum + c.vote_count, 0);
    loading = false;
  });
</script>

<div class="min-h-screen bg-gradient-to-br from-purple-50 to-pink-100 py-12 px-4">
  <div class="max-w-4xl mx-auto">
    <h1 class="text-4xl font-bold text-center text-purple-900 mb-8">
      📊 Keputusan Undi
    </h1>

    <div class="bg-white rounded-lg shadow-lg p-8">
      <div class="text-center mb-8">
        <p class="text-gray-600">Jumlah Undi</p>
        <p class="text-4xl font-bold text-purple-600">{totalVotes}</p>
      </div>

      {#if loading}
        <div class="text-center py-8">
          <p class="text-gray-500">Memuatkan data...</p>
        </div>
      {:else}
        <div class="space-y-6">
          {#each candidates as candidate, i}
            <div class="border-b pb-4">
              <div class="flex items-center justify-between mb-2">
                <div class="flex items-center gap-4">
                  <span class="text-2xl font-bold text-gray-400">#{i + 1}</span>
                  <img src={candidate.photo_url} alt={candidate.name} class="w-12 h-12 rounded-full" />
                  <div>
                    <h3 class="font-semibold text-lg">{candidate.name}</h3>
                    <p class="text-sm text-gray-600">{candidate.description}</p>
                  </div>
                </div>
                <div class="text-right">
                  <p class="text-2xl font-bold text-purple-600">{candidate.vote_count}</p>
                  <p class="text-sm text-gray-500">
                    {totalVotes > 0 ? ((candidate.vote_count / totalVotes) * 100).toFixed(1) : 0}%
                  </p>
                </div>
              </div>
              <div class="w-full bg-gray-200 rounded-full h-3">
                <div 
                  class="bg-purple-600 h-3 rounded-full transition-all duration-500"
                  style="width: {totalVotes > 0 ? (candidate.vote_count / totalVotes) * 100 : 0}%"
                ></div>
              </div>
            </div>
          {/each}
        </div>
      {/if}

      <div class="mt-8 text-center">
        <a href="/" class="inline-block bg-purple-600 text-white px-6 py-3 rounded-lg font-semibold hover:bg-purple-700 transition">
          Kembali ke Halaman Utama
        </a>
      </div>
    </div>
  </div>
</div>