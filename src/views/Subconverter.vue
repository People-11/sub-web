<template>
<div>
    <el-row style="margin-top: 10px">
        <el-col>
            <el-card>
                <div slot="header">
                SubConverter
                    <div style="font-style: normal; font-size: 80%; text-align: right; margin-top: 5px;">
                        {{ backendVersion }}
                    </div>
                </div>
                <el-container>
                    <el-form :model="form" label-width="80px" label-position="left" style="width: 100%">
                        <el-form-item label="模式设置:">
                            <el-radio v-model="advanced" label="1">基础模式</el-radio>
                            <el-radio v-model="advanced" label="2">进阶模式</el-radio>
                        </el-form-item>
                        <el-form-item label="订阅链接:">
                            <el-input v-model="form.sourceSubUrl" style="width: 100%" type="textarea" rows="4" placeholder="支持各种订阅链接或单节点链接，多个链接每行一个或用 | 分隔" @blur="saveSubUrl" />
                        </el-form-item>
                        <el-form-item label="客户端项:">
                            <el-select v-model="form.clientType" style="width: 100%">
                                <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="远程配置:">
                            <el-select v-model="form.remoteConfig" style="width: 100%" allow-create filterable placeholder="请选择，或手动输入远程配置地址">
                                <el-option-group v-for="group in options.remoteConfig" :key="group.label" :label="group.label">
                                    <el-option v-for="item in group.options" :key="item.value" :label="item.label" :value="item.value"></el-option>
                                </el-option-group>
                            </el-select>
                        </el-form-item>
                        <el-form-item label="后端地址:">
                            <el-select v-model="form.customBackend" style="width: 100%" allow-create filterable placeholder="请选择，或手动输入，需要在域名后加/sub?">
                                <el-option v-for="(v, k) in options.customBackend" :key="k" :label="k" :value="v"></el-option>
                            </el-select>
                        </el-form-item>
                        <div v-if="advanced === '2'">
                            <el-form-item label="包含节点:">
                                <el-input v-model="form.includeRemarks" style="width: 100%" placeholder="节点名包含的关键字，支持正则" />
                            </el-form-item>
                            <el-form-item label="排除节点:">
                                <el-input v-model="form.excludeRemarks" style="width: 100%" placeholder="节点名不包含的关键字，支持正则" />
                            </el-form-item>
                            <el-form-item label="输出文件名:">
                                <el-input v-model="form.filename" style="width: 100%" placeholder="返回的订阅文件名" />
                            </el-form-item>
                            <el-form-item label="TUN & DNS:">
                                <el-input v-model="form.clashdns" style="width: 100%" placeholder="tap | win-tun | linux-tun | meta-tun (Clash for Windows适用)" />
                            </el-form-item>
                            <el-form-item label="远程设备:">
                                <el-input v-model="form.devid" placeholder="用于设置QuantumultX的远程设备ID" />
                            </el-form-item>
                            <el-form-item style="width: 100%">
                                <el-row type="flex">
                                    <el-popover placement="bottom" v-model="form.extraset" style="text-align: center">
                                        <el-row :gutter="10">
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.new_name" label="Clash新字段"></el-checkbox>
                                            </el-col>
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.emoji" label="Emoji"></el-checkbox>
                                            </el-col>
                                        </el-row>
                                        <el-row :gutter="10">
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.udp" label="启用 UDP"></el-checkbox>
                                            </el-col>
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.appendType" label="节点类型"></el-checkbox>
                                            </el-col>
                                        </el-row>
                                        <el-row :gutter="10">
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.sort" label="排序节点"></el-checkbox>
                                            </el-col>
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.fdn" label="过滤非法节点"></el-checkbox>
                                            </el-col>
                                        </el-row>
                                        <el-row :gutter="10">
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.tfo" label="TCP Fast Open"></el-checkbox>
                                            </el-col>
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.scv" label="Skip Cert Verify"></el-checkbox>
                                            </el-col>
                                        </el-row>
                                        <el-row :gutter="10">
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.nodeList" label="输出为 Node List"></el-checkbox>
                                            </el-col>
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.tpl.singbox.ipv6" label="Sing-Box支持IPV6"></el-checkbox>
                                            </el-col>
                                        </el-row>
                                        <el-row :gutter="10">
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.tls13" label="开启TLS_1.3"></el-checkbox>
                                            </el-col>
                                            <el-col :span="12">
                                                <el-checkbox v-model="form.xudp" label="启用 XUDP"></el-checkbox>
                                            </el-col>
                                        </el-row>
                                        <el-button slot="reference">节点处理</el-button>
                                    </el-popover>
                                    <el-popover placement="bottom" v-model="form.rule" style="text-align: center">
                                        <el-row :gutter="10">
                                            <el-checkbox v-model="form.expand" label="展开规则"></el-checkbox>
                                        </el-row>
                                        <el-row :gutter="10">
                                            <el-checkbox v-model="form.classic" label="Classic Rule Provider"></el-checkbox>
                                        </el-row>
                                        <el-button slot="reference">Rule Provider 选项</el-button>
                                    </el-popover>
                                </el-row>
                            </el-form-item>
                        </div>
                        <div style="margin-top: 50px"></div>
                        <el-divider content-position="center">
                            <i class="el-icon-magic-stick"></i>
                        </el-divider>
                        <el-form-item label="定制订阅:">
                            <el-input class="copy-content" disabled v-model="customSubUrl">
                                <el-button slot="append" v-clipboard:copy="customSubUrl" v-clipboard:success="onCopy" ref="copy-btn" icon="el-icon-document-copy">复制</el-button>
                            </el-input>
                        </el-form-item>
                        <el-form-item style="margin-top: 40px; text-align: center">
                            <el-button style="width: 120px" type="danger" @click="makeUrl" :disabled="form.sourceSubUrl.length === 0">生成订阅链接</el-button>
                            <el-button style="width: 120px" type="primary" @click="clashInstall" icon="el-icon-connection" :disabled="customSubUrl.length === 0">一键导入Clash</el-button>
                            <!-- <el-button style="width: 120px" type="primary" @click="surgeInstall" icon="el-icon-connection">一键导入Surge</el-button> -->
                        </el-form-item>
                        <el-form-item style="text-align: center">
                            <el-button style="width: 250px" type="primary" @click="dialogLoadConfigVisible = true" icon="el-icon-copy-document" :loading="loading">从URL解析</el-button>
                        </el-form-item>
                    </el-form>
                </el-container>
            </el-card>
        </el-col>
    </el-row>
    <el-dialog :visible.sync="dialogLoadConfigVisible" :show-close="false" :close-on-click-modal="false" :close-on-press-escape="false" style="width: 100%">
        <div slot="title">可以从老的订阅信息中解析信息,填入页面中去</div>
        <el-form label-position="left" style="width: 100%">
            <el-form-item prop="uploadConfig">
                <el-input v-model="loadConfig" type="textarea" :autosize="{ minRows: 15, maxRows: 30 }" maxlength="10000" show-word-limit></el-input>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="loadConfig = ''; dialogLoadConfigVisible = false">取 消</el-button>
            <el-button type="primary" @click="confirmLoadConfig" :disabled="loadConfig.length === 0">确 定</el-button>
        </div>
    </el-dialog>
</div>
</template>

<script>
const defaultBackend =
    process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND + "/sub?";
const tgBotLink = process.env.VUE_APP_BOT_LINK;
export default {
    data() {
        var data = {
            backendVersion: "",
            advanced: "1",
            // 是否为 PC 端
            isPC: true,
            options: {
                clientTypes: {
                    Clash: "clash",
                    Surge: "surge&ver=4",
                    Surge3: "surge&ver=3",
                    Quantumult: "quan",
                    QuantumultX: "quanx",
                    Singbox: "singbox",
                    Loon: "loon",
                    Surfboard: "surfboard",
                    "Shadowsocks(SIP002)": "ss",
                    "Shadowsocks(SIP008)": "sssub",
                    Mellow: "mellow",
                    ShadowsocksR: "ssr",
                    ShadowsocksD: "ssd",
                    Surge2: "surge&ver=2",
                    V2Ray: "v2ray",
                    Trojan: "trojan",
                    "混合订阅（mixed）": "mixed",
                    自动判断客户端: "auto",
                },
                customBackend: {
                    "People11": "https://sub.people11.dev/sub?",
                    "localhost:25500/sub? 本地版": "http://localhost:25500/sub?",
                },
                remoteConfig: [{
                        label: "默认",
                        options: [{ label: "不选，由接口提供方提供", value: "" }],
                    },
                    {
                        label: "自用",
                        options: [{
                                label: "Mixed",
                                value: "https://gist.githubusercontent.com/People-11/4abfa476c6e63918aeecd62c9a206f47/raw/Mixed.ini",
                            },
                        ],
                    },
                ],
            },
            form: {
                appendType: false,
                clashdns: "", // 选择base.tpl 里面Clash DNS区块
                classic: false, // 是否使用 Clash Classic Rule Provider
                clientType: "",
                customBackend: "",
                emoji: false,
                excludeRemarks: "",
                expand: true, // 是否展开规则
                extraset: false,
                tls13: false,
                fdn: false,
                filename: "",
                devid: "",
                includeRemarks: "",
                insert: false, // 是否插入默认订阅的节点，对应配置项 insert_url
                new_name: true, // 是否使用 Clash 新字段
                nodeList: false,
                remoteConfig: "",
                rule: false,
                scv: false,
                sort: false,
                sourceSubUrl: "",
                tfo: false,
                udp: true,
                xudp: false,
                tpl: {
                    singbox: {
                        ipv6: false,
                    },
                },
            },
            loading: false,
            customSubUrl: "",
            loadConfig: "",
            dialogLoadConfigVisible: false,
            myBot: tgBotLink,
        };
        // window.console.log(data.options.remoteConfig);
        // window.console.log(data.options.customBackend);
        let phoneUserAgent = /Android|webOS|iPhone|iPod|BlackBerry|IEMobile|Opera Mini/i.test (navigator.userAgent);
        if (phoneUserAgent) {
            let acl4ssrConfig = data.options.remoteConfig[1].options;
            for (let i = 0; i < acl4ssrConfig.length; i++) {
                if (acl4ssrConfig[i].label.length > 10) {
                    acl4ssrConfig[i].label = acl4ssrConfig[i].label.replace(/\s.*/, "");
                }
            }
            var serverList = {};
            let serverKeys = Object.keys(data.options.customBackend);
            for (let i = 0; i < serverKeys.length; i++) {
                let key = serverKeys[i].replace(/\(.*/, "");
                serverList[key] = data.options.customBackend[serverKeys[i]];
            }
            data.options.customBackend = serverList;
        }
        return data;
    },
    created() {
        // document.title = "Subscription Converter";
        this.isPC = this.$getOS().isPc;
        // 获取 url cache
        if (process.env.VUE_APP_USE_STORAGE === "true") {
            this.form.sourceSubUrl = this.getLocalStorageItem("sourceSubUrl");
        }
    },
    mounted() {
        this.form.clientType = "clash";
        this.form.customBackend = defaultBackend;
        this.form.remoteConfig = "https://gist.githubusercontent.com/People-11/4abfa476c6e63918aeecd62c9a206f47/raw/Mixed.ini";
        this.getBackendVersion();
    },
    methods: {
        onCopy() {
            this.$message.success("Copied!");
        },
        clashInstall() {
            if (this.customSubUrl === "") {
                this.$message.error("请先填写必填项，生成订阅链接");
                return false;
            }
            const url = "clash://install-config?url=";
            window.open(
                url +
                encodeURIComponent(
                    this.curtomShortSubUrl !== "" ?
                    this.curtomShortSubUrl :
                    this.customSubUrl
                )
            );
        },
        surgeInstall() {
            if (this.customSubUrl === "") {
                this.$message.error("请先填写必填项，生成订阅链接");
                return false;
            }
            const url = "surge://install-config?url=";
            window.open(url + this.customSubUrl);
        },
        makeUrl() {
            if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
                this.$message.error("订阅链接与客户端为必填项");
                return false;
            }
            // 远程接口
            let backend =
                this.form.customBackend === "" ?
                defaultBackend :
                this.form.customBackend;
            // 远程配置
            let config = this.form.remoteConfig === "" ? "" : this.form.remoteConfig;
            let sourceSub = this.form.sourceSubUrl;
            sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");
            // 薯条屏蔽
            if (
                sourceSub.indexOf("losadhwse") !== -1 &&
                (backend.indexOf("py6.pw") !== -1 ||
                    backend.indexOf("subconverter-web.now.sh") !== -1 ||
                    backend.indexOf("api.wcc.best") !== -1)
            ) {
                this.$alert(
                    "此机场订阅已将该后端屏蔽，请自建后端转换。",
                    "转换错误提示", {
                        confirmButtonText: "确定",
                        callback: (action) => {
                            this.$message({ type: "error", message: `action: ${action}` });
                        },
                    }
                );
                return false;
            }
            this.customSubUrl =
                backend +
                "target=" +
                this.form.clientType +
                "&url=" +
                encodeURIComponent(sourceSub);
            if (config) {
                this.customSubUrl += "&config=" + encodeURIComponent(config);
            }
            if (this.advanced === "2") {
                if (this.form.insert) {
                    this.customSubUrl +=
                        "&insert=" + this.form.insert;
                }
                if (this.form.excludeRemarks) {
                    this.customSubUrl +=
                        "&exclude=" + encodeURIComponent(this.form.excludeRemarks);
                }
                if (this.form.includeRemarks) {
                    this.customSubUrl +=
                        "&include=" + encodeURIComponent(this.form.includeRemarks);
                }
                if (this.form.filename) {
                    this.customSubUrl +=
                        "&filename=" + encodeURIComponent(this.form.filename);
                }
                if (this.form.devid) {
                    this.customSubUrl += "&dev_id=" + encodeURIComponent(this.form.devid);
                }
                if (this.form.appendType) {
                    this.customSubUrl +=
                        "&append_type=" + this.form.appendType.toString();
                }
                if (this.form.clashdns) {
                    this.customSubUrl +=
                        "&clash.dns=" + encodeURIComponent(this.form.clashdns);
                }
                if (this.form.tls13) {
                    this.customSubUrl += "&tls13=" + this.form.tls13.toString();
                }
                if (this.form.clientType === "clash") {
                    this.customSubUrl += "&new_name=" + this.form.new_name.toString();
                }
                if (this.form.clientType === "singbox") {
                    if (this.form.tpl.singbox.ipv6 === true) {
                        this.customSubUrl += "&singbox.ipv6=1";
                    }
                }
                if (this.form.sort) {
                    this.customSubUrl += "&sort=" + this.form.sort.toString();
                }
                this.customSubUrl +=
                    "&emoji=" +
                    this.form.emoji.toString() +
                    "&list=" +
                    this.form.nodeList.toString() +
                    "&xudp=" +
                    this.form.xudp.toString() +
                    "&udp=" +
                    this.form.udp.toString() +
                    "&tfo=" +
                    this.form.tfo.toString() +
                    "&scv=" +
                    this.form.scv.toString() +
                    "&fdn=" +
                    this.form.fdn.toString() +
                    "&expand=" +
                    this.form.expand.toString();
                if (this.form.classic) {
                    this.customSubUrl += "&classic=" + this.form.classic.toString();
                }
            }
            this.$copyText(this.customSubUrl);
            this.$message.success("定制订阅已复制到剪贴板");
        },
        notify() {
            const h = this.$createElement;
            this.$notify({
                title: "隐私提示",
                type: "warning",
                message: h(
                    "i", { style: "color: teal" },
                    "各种订阅链接（短链接服务除外）生成纯前端实现，无隐私问题。默认提供后端转换服务，隐私担忧者请自行搭建后端服务。"
                ),
            });
        },
        confirmLoadConfig() {
            // 怎么解析短链接的302和301...
            if (this.loadConfig.indexOf("target") === -1) {
                this.$message.error("请输入正确的订阅地址,暂不支持短链接!");
                return;
            }
            let url;
            try {
                url = new URL(this.loadConfig);
            } catch (error) {
                this.$message.error("请输入正确的订阅地址!");
                return;
            }
            this.form.customBackend = url.origin + url.pathname + "?";
            let param = new URLSearchParams(url.search);
            if (param.get("target")) {
                let target = param.get("target");
                if (target === "surge" && param.get("ver")) {
                    // 类型为surge,有ver
                    this.form.clientType = target + "&ver=" + param.get("ver");
                } else if (target === "surge") {
                    //类型为surge,没有ver
                    this.form.clientType = target + "&ver=4";
                } else {
                    //类型为其他
                    this.form.clientType = target;
                }
            }
            if (param.get("url")) {
                this.form.sourceSubUrl = param.get("url");
            }
            if (param.get("insert")) {
                this.form.insert = param.get("insert") === "true";
            }
            if (param.get("config")) {
                this.form.remoteConfig = param.get("config");
            }
            if (param.get("exclude")) {
                this.form.excludeRemarks = param.get("exclude");
            }
            if (param.get("include")) {
                this.form.includeRemarks = param.get("include");
            }
            if (param.get("filename")) {
                this.form.filename = param.get("filename");
            }
            if (param.get("dev_id")) {
                this.form.devid = param.get("dev_id");
            }
            if (param.get("append_type")) {
                this.form.appendType = param.get("append_type") === "true";
            }
            if (param.get("tls13")) {
                this.form.tls13 = param.get("tls13");
            }
            if (param.get("xudp")) {
                this.form.xudp = param.get("xudp") === "true";
            }
            if (param.get("sort")) {
                this.form.sort = param.get("sort") === "true";
            }
            if (param.get("emoji")) {
                this.form.emoji = param.get("emoji") === "true";
            }
            if (param.get("list")) {
                this.form.nodeList = param.get("list") === "true";
            }
            if (param.get("udp")) {
                this.form.udp = param.get("udp") === "true";
            }
            if (param.get("tfo")) {
                this.form.tfo = param.get("tfo") === "true";
            }
            if (param.get("expand")) {
                this.form.expand = param.get("expand") === "true";
            }
            if (param.get("scv")) {
                this.form.scv = param.get("scv") === "true";
            }
            if (param.get("fdn")) {
                this.form.fdn = param.get("fdn") === "true";
            }
            if (param.get("clash.dns")) {
                this.form.clashdns = param.get("clash.dns");
            }
            if (param.get("classic")) {
                this.form.classic = param.get("classic") === "false";
            }
            if (param.get("new_name")) {
                this.form.new_name = param.get("new_name") === "true";
            }
            this.dialogLoadConfigVisible = false;
        },
        renderPost() {
            let data = new FormData();
            data.append("target", encodeURIComponent(this.form.clientType));
            data.append("url", encodeURIComponent(this.form.sourceSubUrl));
            data.append("config", encodeURIComponent(this.form.remoteConfig));
            data.append("exclude", encodeURIComponent(this.form.excludeRemarks));
            data.append("include", encodeURIComponent(this.form.includeRemarks));
            data.append("tls13", encodeURIComponent(this.form.tls13.toString()));
            data.append("xudp", encodeURIComponent(this.form.xudp.toString()));
            data.append("emoji", encodeURIComponent(this.form.emoji.toString()));
            data.append("list", encodeURIComponent(this.form.nodeList.toString()));
            data.append("udp", encodeURIComponent(this.form.udp.toString()));
            data.append("tfo", encodeURIComponent(this.form.tfo.toString()));
            data.append("expand", encodeURIComponent(this.form.expand.toString()));
            data.append("scv", encodeURIComponent(this.form.scv.toString()));
            data.append("fdn", encodeURIComponent(this.form.fdn.toString()));
            data.append(
                "clash.dns",
                encodeURIComponent(this.form.clashdns.toString())
            );
            data.append("newname", encodeURIComponent(this.form.new_name.toString()));
            return data;
        },
        createFilter(queryString) {
            return (candidate) => {
                return (
                    candidate.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
                );
            };
        },
        getBackendVersion() {
            this.$axios
                .get(
                    defaultBackend.substring(0, defaultBackend.length - 5) + "/version"
                )
                .then((res) => {
                    this.backendVersion = res.data.replace(/backend\n$/gm, "");
                    this.backendVersion = this.backendVersion.replace("subconverter", "");
                });
        },
        saveSubUrl() {
            if (this.form.sourceSubUrl !== "") {
                this.setLocalStorageItem("sourceSubUrl", this.form.sourceSubUrl);
            }
        },
        getLocalStorageItem(itemKey) {
            const now = +new Date();
            let ls = localStorage.getItem(itemKey);
            let itemValue = "";
            if (ls !== null) {
                let data = JSON.parse(ls);
                if (data.expire > now) {
                    itemValue = data.value;
                } else {
                    localStorage.removeItem(itemKey);
                }
            }
            return itemValue;
        },
        setLocalStorageItem(itemKey, itemValue) {
            const ttl = process.env.VUE_APP_CACHE_TTL;
            const now = +new Date();
            let data = {
                setTime: now,
                ttl: parseInt(ttl),
                expire: now + ttl * 1000,
                value: itemValue,
            };
            localStorage.setItem(itemKey, JSON.stringify(data));
        },
    },
};
</script>
