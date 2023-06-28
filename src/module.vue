<template>
	<private-view :title="page_title">
		<template v-if="breadcrumb" #headline>
			<v-breadcrumb :items="breadcrumb" />
		</template>

		<template #navigation>
			<page-navigation :current="page" :pages="all_pages"/>
		</template>

		<div class="lp-container">
			<div class="lp-banner" v-if="page_banner">
				<img :src="page_banner" alt=""/>
			</div>
			<div class="lp-cards" v-if="page_cards">
				<div class="lp-card" v-for="card in page_cards.filter(item => (item.uri != page))" :key="card.uri" :style="`background-color: ${card.color}`" @click="change_page(card.to)">
					<v-icon :name="card.icon"/>
					<span class="lp-card-title">{{ card.label }}</span>
				</div>
			</div>
			<div class="lp-body" v-if="page_body" v-html="page_body"></div>
		</div>
		
		<router-view name="landing-page" :page="page" />
	</private-view>
</template>

<script>
import { ref, watch } from 'vue';
import { useApi } from '@directus/extensions-sdk';
import { useRouter } from 'vue-router';
import PageNavigation from './components/navigation.vue';

export default {
	components: {
		PageNavigation,
	},
	props: {
		page: {
			type: String,
			default: 'home',
		},
	},
	setup(props) {
		const router = useRouter();
		const api = useApi();
		const { addTokenToURL } = useDirectusToken(api);
		const page_title = ref('');
		const page_banner = ref('');
		const page_cards = ref([]);
		const page_body = ref('');
		const breadcrumb = ref([
            {
                name: 'Home',
                to: `/landing-page`,
            },
        ]);
		const all_pages = ref([]);

		render_page(props.page);
		fetch_all_pages();

		watch(
            () => props.page,
            () => {
                render_page(props.page);
            }
        );

		function change_page(to){
			const next = router.resolve(`${to}`);
			router.push(next);
		}

		return { page_title, page_banner, page_cards, page_body, breadcrumb, all_pages, change_page };

		function render_page(page){
			if(page === null){
				page_title.value = '500: Internal Server Error';
				breadcrumb.value.splice(1, 1);
				page_banner.value = '';
				page_cards.value = [];
				page_body.value = '';
			} else {
				switch(page) {
					case 'home':
						page_title.value = 'Home';
						page_banner.value = addTokenToURL('/assets/83BD365C-C3CE-4015-B2AD-63EDA9E52A69?width=2000&height=563&fit=cover');
						page_cards.value = all_pages.value;
						page_body.value = '<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>';
						break;
					case 'hello-world':
						page_title.value = 'Hello World';
						page_banner.value = addTokenToURL('/assets/853B243D-A1BF-6051-B1BF-23EDA8E32A09?width=2000&height=563&fit=cover');
						page_cards.value = all_pages.value;
						page_body.value = '<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>';
						break;
					case 'contact':
						page_title.value = 'Contact Us';
						page_banner.value = addTokenToURL('/assets/91CE173D-A1AD-4104-A1EC-74FCB8F41B58?width=2000&height=563&fit=cover');
						page_cards.value = [];
						page_body.value = '<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.</p>';
						break;
					default:
						page_title.value = '404: Not Found';
				}

				if(page === 'home'){
					breadcrumb.value.splice(1, 1);
				} else {
					breadcrumb.value[1] = {
						name: page_title.value,
						to: `/landing-page/${page}`,
					};
				}
			}
		}

		// Using the API
		// function render_page(page){
		// 	if(page === null){
		// 		page_title.value = '500: Internal Server Error';
		// 		breadcrumb.value.splice(1, 1);
		// 	} else {
		// 		api.get(`/items/pages?fields=title,banner,content&filter[uri][_eq]=${page}`).then((rsp) => {
		// 			if(rsp.data.data){
		// 				rsp.data.data.forEach(item => {
		// 					page_title.value = item.title;
        // 					page_banner.value = addTokenToURL(`/assets/${item.banner}?width=2000&height=563&fit=cover`);
        // 					page_body.value = item.content;
		// 				});
		//				page_cards.value = all_pages.value;
		// 			} else {
		// 				page_title.value = "404: Not Found";
		// 			}
		// 		}).catch((error) => {
		// 			console.log(error);
		// 		});
		// 		if(page === 'home'){
		// 			breadcrumb.value.splice(1, 1);
		// 		} else {
		// 			breadcrumb.value[1] = {
		// 				name: page_title.value,
		// 				to: `/landing-page/${page}`,
		// 			};
		// 		}
		// 	}

		// 	console.log(`Title: ${page_title.value}`);
		// }

		function fetch_all_pages(){
			all_pages.value = [
				{
					label: 'Home',
					uri: 'landing-page',
					to: '/landing-page',
					icon: 'home',
					color: '#6644FF',
				},
				{
					label: 'Hello World',
					uri: 'hello-world',
					to: '/landing-page/hello-world',
					icon: 'public',
					color: '#2ECDA7',
				},
				{
					label: 'Contact Us',
					uri: 'contact',
					to: '/landing-page/contact',
					icon: 'phone',
					color: '#3399FF',
				},
			];
			console.log(all_pages.value);
		}

		// Using the API
		// function fetch_all_pages(){
		// 	api.get('/items/pages?fields=title,uri,icon,color').then((rsp) => {
		// 		all_pages.value = [];
		// 		rsp.data.data.forEach(item => {
		// 			all_pages.value.push({
		// 				label: item.title,
		// 				uri: item.uri,
		// 				to: `/landing-page/${item.uri}`,
		// 				icon: item.icon,
		// 				color: item.color,
		// 			});
		// 		});
		// 	}).catch((error) => {
        //         console.log(error);
        //     });
		// }
	},
};
</script>
<style lang="scss">
	.lp-container {
		padding: var(--content-padding);
		padding-top: 0;
		width: 100%;
		max-width: 1024px;

		&> div {
			margin-bottom: var(--content-padding);
		}
	}

	.lp-banner {
		border-radius: var(--border-radius);
		overflow: hidden;

		img {
			display: block;
			width: 100%;
		}
	}

	.lp-cards {
		display: grid;
		grid-template-columns: repeat(4, 1fr);
		column-gap: var(--input-padding);
    	row-gap: var(--input-padding);

		.lp-card {
			display: flex;
			flex-wrap: wrap;
			align-items: center;
			justify-content: center;
			text-align: center;
			border-radius: var(--border-radius);
			padding: var(--input-padding);
			color: white;

			.v-icon {
				width: 100%;
				height: 50px;
				margin-bottom: 6px;

				i {
					font-size: 50px;
    				color: white;
				}
			}

			.lp-card-title {
				display: block;
				font-weight: bold;
				font-size: 1.4em;
				line-height: 1.2;
			}
		}
	}
</style>