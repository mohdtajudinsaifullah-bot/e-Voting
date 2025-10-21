<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';

  let email = '';
  let fullName = '';
  let step = 'register'; // register, vote, success
  let candidates = [];
  let selectedCandidate = null;
  let loading = false;
  let error = '';
  let voterId = null;

  onMount(async () => {
    await loadCandidates();
  });

  async function loadCandidates() {
  console.log('🔍 Loading candidates...');
  
  const { data, error: err } = await supabase
    .from('candidates')
    .select('*')
    .order('name');
  
  console.log('📊 Supabase response:', { data, err });
  
  if (err) {
    error = 'Gagal load calon: ' + err.message;
    console.log('❌ Error:', err);
  } else {
    candidates = data;
    console.log('✅ Candidates loaded:', candidates);
  }
}

  async function registerVoter() {
    if (!email || !fullName) {
      error = 'Sila isi semua maklumat';
      return;
    }

    loading = true;
    error = '';

    const { data: existing } = await supabase
      .from('voters')
      .select('*')
      .eq('email', email)
      .single();

    if (existing) {
      if (existing.has_voted) {
        error = 'Anda sudah mengundi!';
        loading = false;
        return;
      }
      voterId = existing.id;
      step = 'vote';
      loading = false;
      return;
    }

    const { data, error: err } = await supabase
      .from('voters')
      .insert([{ email, full_name: fullName }])
      .select()
      .single();

    loading = false;

    if (err) {
      error = 'Gagal daftar: ' + err.message;
    } else {
      voterId = data.id;
      step = 'vote';
    }
  }

  async function submitVote() {
    if (!selectedCandidate) {
      error = 'Sila pilih calon';
      return;
    }

    loading = true;
    error = '';

    const { error: voteErr } = await supabase
      .from('votes')
      .insert([{ voter_id: voterId, candidate_id: selectedCandidate }]);

    if (voteErr) {
      error = 'Gagal mengundi: ' + voteErr.message;
      loading = false;
      return;
    }

    const { error: voterErr } = await supabase
      .from('voters')
      .update({ has_voted: true, voted_at: new Date().toISOString() })
      .eq('id', voterId);

    if (voterErr) {
      error = 'Gagal update status: ' + voterErr.message;
      loading = false;
      return;
    }

    const candidate = candidates.find(c => c.id === selectedCandidate);
    const { error: countErr } = await supabase
      .from('candidates')
      .update({ vote_count: candidate.vote_count + 1 })
      .eq('id', selectedCandidate);

    loading = false;

    if (countErr) {
      error = 'Gagal update count: ' + countErr.message;
    } else {
      step = 'success';
    }
  }
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 py-12 px-4">
  <div class="max-w-4xl mx-auto">
    <h1 class="text-4xl font-bold text-center text-indigo-900 mb-8">
      🗳️ Sistem E-Voting
    </h1>

    {#if error}
      <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
        {error}
      </div>
    {/if}

    {#if step === 'register'}
      <div class="bg-white rounded-lg shadow-lg p-8">
        <h2 class="text-2xl font-semibold mb-6 text-gray-800">Pendaftaran Pengundi</h2>
        
        <div class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Nama Penuh</label>
            <input
              type="text"
              bind:value={fullName}
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
              placeholder="Nama penuh anda"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-gray-700 mb-2">Email</label>
            <input
              type="email"
              bind:value={email}
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-transparent"
              placeholder="email@example.com"
            />
          </div>

          <button
            on:click={registerVoter}
            disabled={loading}
            class="w-full bg-indigo-600 text-white py-3 rounded-lg font-semibold hover:bg-indigo-700 disabled:opacity-50 disabled:cursor-not-allowed transition"
          >
            {loading ? 'Memproses...' : 'Teruskan ke Undi'}
          </button>
        </div>
      </div>
    {/if}

    {#if step === 'vote'}
      <div class="bg-white rounded-lg shadow-lg p-8">
        <h2 class="text-2xl font-semibold mb-6 text-gray-800">Pilih Calon Anda</h2>
        
        <div class="grid gap-6 md:grid-cols-2 lg:grid-cols-3 mb-6">
          {#each candidates as candidate}
            <button
              on:click={() => selectedCandidate = candidate.id}
              class="p-6 border-2 rounded-lg transition hover:shadow-lg {selectedCandidate === candidate.id ? 'border-indigo-600 bg-indigo-50' : 'border-gray-200'}"
            >
              <img src={candidate.photo_url} alt={candidate.name} class="w-24 h-24 rounded-full mx-auto mb-4" />
              <h3 class="font-semibold text-lg text-gray-800">{candidate.name}</h3>
              <p class="text-sm text-gray-600 mt-2">{candidate.description}</p>
            </button>
          {/each}
        </div>

        <button
          on:click={submitVote}
          disabled={loading || !selectedCandidate}
          class="w-full bg-green-600 text-white py-3 rounded-lg font-semibold hover:bg-green-700 disabled:opacity-50 disabled:cursor-not-allowed transition"
        >
          {loading ? 'Menghantar...' : 'Hantar Undi'}
        </button>
      </div>
    {/if}

    {#if step === 'success'}
      <div class="bg-white rounded-lg shadow-lg p-8 text-center">
        <div class="text-6xl mb-4">✅</div>
        <h2 class="text-3xl font-bold text-green-600 mb-4">Terima Kasih!</h2>
        <p class="text-gray-600 text-lg">Undi anda telah berjaya direkodkan.</p>
        <a href="/results" class="inline-block mt-6 bg-indigo-600 text-white px-6 py-3 rounded-lg font-semibold hover:bg-indigo-700">
          Lihat Keputusan
        </a>
      </div>
    {/if}
  </div>
</div>