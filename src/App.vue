<script setup lang="ts">
    import { onMounted, ref, watch, computed } from "vue";
    import Header from "./components/util/Header.vue";
    import Footer from "./components/util/Footer.vue";
    import Alert from "./components/util/Alert.vue";
    import { useStore } from "./store/store";
    import { LeagueInfoType } from "./api/types";
    import { inject } from "@vercel/analytics";

    const store = useStore();
    const systemDarkMode = window.matchMedia(
        "(prefers-color-scheme: dark)"
    ).matches;
    const clicked = ref(systemDarkMode);

    onMounted(() => {
        if (systemDarkMode && !localStorage.darkMode) {
            clicked.value = true;
            store.updateDarkMode(true);
        } else if (localStorage.darkMode) {
            clicked.value = JSON.parse(localStorage.darkMode);
            store.updateDarkMode(clicked.value);
        }
    });

    watch(clicked, () => {
        localStorage.darkMode = clicked.value;
        store.updateDarkMode(clicked.value);
    });

    const darkMode = computed(() => {
        return store.darkMode;
    });

    const setColorMode = () => {
        clicked.value = !clicked.value;
    };

    onMounted(async () => {
        inject();
        setHtmlBackground();
    });

    watch(
        () => store.darkMode,
        () => setHtmlBackground()
    );

    watch(
        () => store.currentLeagueId,
        () => {
            if (store.currentLeagueId === "") {
                localStorage.removeItem("currentLeagueId");
                localStorage.removeItem("leagueInfo");
            } else {
                localStorage.currentLeagueId = store.currentLeagueId;
                // update league id in url
                // sometimes errors to undefined, TODO
                if (store.currentLeagueId !== "undefined") {
                    const url: any = new URL(window.location.href);
                    url.searchParams.set("leagueId", store.currentLeagueId);
                    window.history.pushState({}, "", url.toString());
                }
            }
        }
    );

    watch(
        () => store.leagueInfo.length,
        () => {
            if (
                store.leagueInfo.length > 0 &&
                !store.showRemovedAlert &&
                store.leagueSubmitted
            ) {
                store.updateShowAddedAlert(true);
                setTimeout(() => {
                    store.updateShowAddedAlert(false);
                }, 3000);
                store.leagueSubmitted = false;
            }
            localStorage.setItem(
                "leagueInfo",
                JSON.stringify(store.leagueInfo as LeagueInfoType[])
            );
        }
    );

    const setHtmlBackground = () => {
        const html = document.querySelector("html");
        if (html) {
            if (store.darkMode) {
                html.style.backgroundColor = "#020617";
                document
                    .querySelector('meta[name="theme-color"]')
                    ?.setAttribute("content", "#020617");
            } else {
                html.style.backgroundColor = "#f8fafc";
                document
                    .querySelector('meta[name="theme-color"]')
                    ?.setAttribute("content", "#f8fafc");
            }
        }
    };
</script>

<template>
    <div :class="{ dark: store.darkMode }" class="h-screen">
        <div class="h-full bg-slate-50 dark:bg-slate-950">
            <Header />
            <button aria-label="Button to toggle dark mode"
                    @click="setColorMode()"
                    data-tooltip-target="tooltip-bottom"
                    data-tooltip-placement="bottom"
                    type="button"
                    class="absolute top-4 right-4 text-white bg-slate-50 hover:bg-slate-300 focus:ring focus:outline-none focus:ring-slate-300 font-small rounded-lg text-sm p-2.5 text-center inline-flex items-center me-2 dark:bg-slate-950 dark:hover:bg-gray-600 dark:focus:ring-gray-600">
                <svg v-if="darkMode"
                     class="w-5 h-5 text-gray-800 dark:text-white"
                     aria-hidden="true"
                     xmlns="http://www.w3.org/2000/svg"
                     fill="currentColor"
                     viewBox="0 0 20 20">
                    <path d="M10 15a5 5 0 1 0 0-10 5 5 0 0 0 0 10Zm0-11a1 1 0 0 0 1-1V1a1 1 0 0 0-2 0v2a1 1 0 0 0 1 1Zm0 12a1 1 0 0 0-1 1v2a1 1 0 1 0 2 0v-2a1 1 0 0 0-1-1ZM4.343 5.757a1 1 0 0 0 1.414-1.414L4.343 2.929a1 1 0 0 0-1.414 1.414l1.414 1.414Zm11.314 8.486a1 1 0 0 0-1.414 1.414l1.414 1.414a1 1 0 0 0 1.414-1.414l-1.414-1.414ZM4 10a1 1 0 0 0-1-1H1a1 1 0 0 0 0 2h2a1 1 0 0 0 1-1Zm15-1h-2a1 1 0 1 0 0 2h2a1 1 0 0 0 0-2ZM4.343 14.243l-1.414 1.414a1 1 0 1 0 1.414 1.414l1.414-1.414a1 1 0 0 0-1.414-1.414ZM14.95 6.05a1 1 0 0 0 .707-.293l1.414-1.414a1 1 0 1 0-1.414-1.414l-1.414 1.414a1 1 0 0 0 .707 1.707Z" />
                </svg>
                <svg v-else
                     class="w-5 h-5 text-gray-800 dark:text-white"
                     aria-hidden="true"
                     xmlns="http://www.w3.org/2000/svg"
                     fill="currentColor"
                     viewBox="0 0 18 20">
                    <path d="M17.8 13.75a1 1 0 0 0-.859-.5A7.488 7.488 0 0 1 10.52 2a1 1 0 0 0 0-.969A1.035 1.035 0 0 0 9.687.5h-.113a9.5 9.5 0 1 0 8.222 14.247 1 1 0 0 0 .004-.997Z" />
                </svg>
            </button>
            <div class="w-full border-b border-slate-200 dark:border-slate-600"></div>
            <RouterView />
            <div class="w-full mt-16 border-b border-slate-200 dark:border-slate-600"></div>
            <!--<Footer />-->
        </div>
        <Alert v-if="store.showAddedAlert"
               alert-msg="League successfully added!"
               type="success" />
        <Alert v-if="store.showRefreshAlert"
               alert-msg="League data refreshed!"
               type="success" />
        <Alert v-if="store.showRemovedAlert"
               alert-msg="League removed!"
               type="success" />
        <Alert v-if="store.showCopiedAlert"
               alert-msg="Link copied to clipboard!"
               type="success" />
        <Alert v-if="store.showCopyReport"
               alert-msg="Summary copied to clipboard!"
               type="success" />
        <Alert v-if="store.showLoadingAlert"
               alert-msg="Error fetching data. Please try refreshing the page."
               type="error" />
    </div>
</template>
<style scoped>
    .absolute {
        position: absolute;
    }

    .top-4 {
        top: 1rem;
    }

    .right-4 {
        right: 1rem;
    }
</style>
