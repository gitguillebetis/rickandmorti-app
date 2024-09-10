<template>
	<div class="container-xl">
		<h1 class="mb-5">Rick and Morty Characters</h1>
		<div class="row justify-content-center mb-3">
			<div class="d-flex p-2 gap-2">
				<div class="d-inline-flex gap-1 w-100">
					<input
						v-model="searchQuery"
						type="text"
						class="form-control"
						placeholder="Search for a character"
						@keydown.enter="searchCharacters"
					/>
				</div>
				<div class="d-inline-flex gap-1">
					<button 
						class="btn btn-primary" 
						@click="searchCharacters">
						Search
					</button>
					<button 
						class="btn btn-secondary"
						@click="resetSearch"
						:disabled="!searchActive">
						Reset
					</button>
				</div>
			</div>
		</div>
		<div class="row justify-content-center mb-3">
			<span class="character-count">{{ totalCharacterCount }} characters found</span>
		</div>	
		<div class="row justify-content-center">
			<div class="col-12">
				<table class="table">
					<thead>
						<tr>
							<th scope="col" class="name">Name</th>
							<th scope="col" class="gender">Gender</th>
							<th scope="col" class="status">Status</th>
							<th scope="col" class="species">Species</th>
							<th scope="col" class="location">Location</th>
							<th scope="col" class="episodes">Episodes</th>
						</tr>
					</thead>
					<tbody v-if="characters.length">
						<tr v-for="character in characters" :key="character.id">
							<td colspan="6" class="accordion" :id="'accordion-' + character.id">
								<table class="table accordion-item">
									<tr class="accordion-header">
										<td class="name">
											<button 
												class="accordion-button collapsed" 
												type="button" 
												:data-bs-toggle="'collapse'"
												:data-bs-target="'#collapse-' + character.id" 
												aria-expanded="false" 
												:aria-controls="'collapse-' + character.id">
												<div>{{ character.name }}</div>
											</button>
										</td>
										<td class="gender">{{ character.gender }}</td>
										<td class="status">{{ character.status }}</td>
										<td class="species">{{ character.species }}</td>
										<td class="location">{{ character.location.name }}</td>
										<td class="episodes">{{ character.episodes?.length || 0 }}</td>
									</tr>
									<tr :id="'collapse-' + character.id" class="accordion-collapse collapse" :data-bs-parent="'#accordion-' + character.id" >
										<td colspan="6" class="accordion-body">
											<table class="table extra">
												<tr>
													<td class="image"><img :src="character.image" alt="Character image" class="character-image" /></td>
													<td class="info">
														<p>
															{{ character.name }}<br>
															{{ character.gender }}<br>
															{{ character.status }}<br>
															{{ character.species }}<br>
															{{ character.origin.name }}<br>
															{{ character.location.name }}
														</p>
													</td>
													<td class="episodes">
														<table class="table">
															<thead>
																<tr>
																	<th scope="col">Episode Name</th>
																	<th scope="col">Season</th>
																	<th scope="col">Episode</th>
																</tr>
															</thead>
															<tbody>
																<tr v-for="episode in character.episodes" :key="episode.id">
																	<td>{{ episode.name }}</td>
																	<td>{{ getSeason(episode.episode) }}</td>
																	<td>{{ getEpisodeNumber(episode.episode) }}</td>
																</tr>
															</tbody>
														</table>
													</td>
												</tr>
											</table>
										</td>
									</tr>
								</table>
							</td>
						</tr>
					</tbody>
					<tfoot v-else>
						<tr>
							<td colspan="5">Loading characters...</td>
						</tr>
					</tfoot>
				</table>
				<nav v-if="pagination">
					<ul class="pagination justify-content-center">
						<li class="page-item" :class="{ disabled: !pagination.prev }">
							<button class="page-link" @click="fetchCharacters(pagination.prev)" :disabled="!pagination.prev">Previous</button>
						</li>
						<li class="page-item disabled">
							<span class="page-link">Page {{ currentPage }} of {{ totalPages }}</span>
						</li>
						<li class="page-item" :class="{ disabled: !pagination.next }">
							<button class="page-link" @click="fetchCharacters(pagination.next)" :disabled="!pagination.next">Next</button>
						</li>
					</ul>
				</nav>
			</div>
		</div>
	</div>
</template>
  
<script>
	import { ref, onMounted } from "vue";

	export default {
		setup() {
			const BASE_URL = "https://rickandmortyapi.com/api/character";

			const characters = ref([]);
			const pagination = ref({ next: null, prev: null });
			const currentPage = ref(1); 
			const totalPages = ref(0);
			const totalCharacterCount = ref(0);
			const searchQuery = ref("");
			const searchActive = ref(false); 

			const getSeason = (episodeCode) => episodeCode.substring(1, 3);
			const getEpisodeNumber = (episodeCode) => episodeCode.substring(4, 6);

			const fetchEpisodes = async (character) => {
				const episodePromises = character.episode.map((url) =>
					fetch(url).then((response) => response.json())
				);
				character.episodes = await Promise.all(episodePromises);
			};

			const fetchCharacters = async (url = BASE_URL) => {
				const response = await fetch(url);
				const data = await response.json();
				characters.value = data.results || [];
				pagination.value.next = data.info.next;
				pagination.value.prev = data.info.prev;
				totalPages.value = data.info.pages;
				totalCharacterCount.value = data.info.count

				for (const character of characters.value) {
					await fetchEpisodes(character);
				}

				const currentUrlParams = new URLSearchParams(url.split('?')[1]);
				currentPage.value = currentUrlParams.has('page') ? parseInt(currentUrlParams.get('page')) : 1;
			};

			const searchCharacters = () => {
				const searchUrl = `${BASE_URL}/?name=${searchQuery.value}`;
				fetchCharacters(searchUrl);
				searchActive.value = true;
			};

			const resetSearch = () => {
				searchQuery.value = "";
				searchActive.value = false;
				fetchCharacters();
			};

			onMounted(fetchCharacters);

			return {
				characters,
				pagination,
				currentPage,
				totalPages,
				totalCharacterCount,
				getSeason,
				getEpisodeNumber,
				fetchCharacters,
				searchQuery,
				searchCharacters,
				resetSearch,
				searchActive
			};
		},
	};
</script>
  
<style scoped>
h1 {
	text-align: center
}
.name {
	width: 20%;
	text-align: left;
}
.gender {
	width: 150px;
}
.status {
	width: 150px;
}
.species {
	width: 20%;
}
.location {
	width: 20%;
}
.episodes {
	width: 100px;
	text-align: center;
}
.accordion {
	--bs-accordion-border-color: none;
}
.accordion-button > div {
	padding-left: 30px;
}
.accordion-button::after {
	position: absolute;
	transform: rotate(-90deg);
}
.accordion-button:not(.collapsed)::after {
	transform: rotate(0deg);
}
.accordion-button:focus {
	box-shadow: none;
}
.accordion-button:not(.collapsed) {
	background-color: transparent
}
.character-image {
	width: 200px;
}
.table thead th {
	padding-left: 0;
	background-color: lightgray;

	&:first-child  {
		padding-left: 20px;
	}

}
.table {
	text-align: left;
}
.table.extra {
	width: 80%;
	td {
		vertical-align: top;
	}
	.image {
		width: 25%;
	}
	.info {
		width: 25%;
	}
	.episodes {
		width: 50%;
	}
}
</style> 