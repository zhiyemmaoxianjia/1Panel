<template>
    <el-drawer v-model="open" :destroy-on-close="true" size="50%">
        <template #header>
            <DrawerHeader :header="$t('app.detail')" :back="handleClose" />
        </template>
        <div class="brief" v-loading="loadingApp">
            <div class="detail flex">
                <div class="w-12 h-12 bg-gray-100 rounded p-1 shadow-md icon">
                    <img :src="app.icon" alt="App Icon" class="w-full h-full rounded" />
                </div>
                <div class="ml-4">
                    <div class="name mb-2">
                        <span>{{ app.name }}</span>
                    </div>
                    <div class="description mb-4">
                        <span>
                            {{ language == 'zh' || language == 'tw' ? app.shortDescZh : app.shortDescEn }}
                        </span>
                    </div>
                    <br />
                    <div v-if="!loadingDetail" class="mb-2">
                        <el-alert
                            style="width: 300px"
                            v-if="!appDetail.enable"
                            :title="$t('app.limitHelper')"
                            type="warning"
                            show-icon
                            :closable="false"
                        />
                    </div>
                    <el-button round v-if="appDetail.enable" @click="openInstall" type="primary">
                        {{ $t('app.install') }}
                    </el-button>
                </div>
            </div>
            <div class="divider"></div>
            <div>
                <el-row>
                    <el-col :span="12">
                        <div class="descriptions">
                            <el-descriptions direction="vertical">
                                <el-descriptions-item>
                                    <el-link @click="toLink(app.website)">
                                        <el-icon><OfficeBuilding /></el-icon>
                                        <span>{{ $t('app.appOfficeWebsite') }}</span>
                                    </el-link>
                                </el-descriptions-item>
                                <el-descriptions-item>
                                    <el-link @click="toLink(app.document)">
                                        <el-icon><Document /></el-icon>
                                        <span>{{ $t('app.document') }}</span>
                                    </el-link>
                                </el-descriptions-item>
                                <el-descriptions-item>
                                    <el-link @click="toLink(app.github)">
                                        <el-icon><Link /></el-icon>
                                        <span>{{ $t('app.github') }}</span>
                                    </el-link>
                                </el-descriptions-item>
                            </el-descriptions>
                        </div>
                    </el-col>
                </el-row>
            </div>
        </div>
        <MdEditor
            previewOnly
            v-model="app.readMe"
            :theme="globalStore.$state.themeConfig.theme === 'dark' ? 'dark' : 'light'"
        />
    </el-drawer>
    <Install ref="installRef"></Install>
</template>

<script lang="ts" setup>
import { GetApp, GetAppDetail } from '@/api/modules/app';
import MdEditor from 'md-editor-v3';
import { ref } from 'vue';
import { useI18n } from 'vue-i18n';
import Install from './install/index.vue';
import router from '@/routers';
import { GlobalStore } from '@/store';
const globalStore = GlobalStore();

const language = useI18n().locale.value;

const app = ref<any>({});
const appDetail = ref<any>({});
const version = ref('');
const loadingDetail = ref(false);
const loadingApp = ref(false);
const installRef = ref();
const open = ref(false);
const appKey = ref();

const acceptParams = async (key: string) => {
    appKey.value = key;
    open.value = true;
    getApp();
};

const handleClose = () => {
    open.value = false;
};

const getApp = async () => {
    loadingApp.value = true;
    try {
        const res = await GetApp(appKey.value);
        app.value = res.data;
        app.value.icon = 'data:image/png;base64,' + res.data.icon;
        version.value = app.value.versions[0];
        getDetail(app.value.id, version.value);
    } finally {
        loadingApp.value = false;
    }
};

const getDetail = async (id: number, version: string) => {
    loadingDetail.value = true;
    try {
        const res = await GetAppDetail(id, version, 'app');
        appDetail.value = res.data;
    } finally {
        loadingDetail.value = false;
    }
};

const toLink = (link: string) => {
    window.open(link, '_blank');
};

const openInstall = () => {
    switch (app.value.type) {
        case 'php':
            router.push({ path: '/websites/runtimes/php' });
            break;
        case 'node':
            router.push({ path: '/websites/runtimes/node' });
            break;
        default:
            const params = {
                app: app.value,
            };
            installRef.value.acceptParams(params);
            open.value = false;
    }
};

defineExpose({
    acceptParams,
});
</script>

<style lang="scss">
.brief {
    padding: 10px;
    .name {
        span {
            font-weight: 500;
            font-size: 18px;
        }
    }

    .description {
        margin-top: 10px;
        span {
            font-size: 14px;
            color: var(--el-text-color-regular);
        }
    }

    .icon {
        width: 180px;
        height: 180px;
    }

    .version {
        margin-top: 10px;
    }

    .descriptions {
        margin-top: 5px;
    }
}
</style>
