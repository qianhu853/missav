// ==UserScript==
// @name         MissAV - MiraPlay 优化版
// @namespace    gmspider
// @version      2026.05.10
// @description  MissAV 专供 MiraPlay 使用（更稳定）
// @author       Grok
// @match        https://missav.*/*
// @match        https://*.missav.*/*
// @require      https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.slim.min.js
// @grant        unsafeWindow
// ==/UserScript==

console.log("🚀 MissAV MiraPlay 优化版 已加载");

(function () {
    const GMSpiderArgs = {};
    if (typeof GmSpiderInject !== 'undefined') {
        let args = JSON.parse(GmSpiderInject.GetSpiderArgs());
        GMSpiderArgs.fName = args.shift();
        GMSpiderArgs.fArgs = args;
    } else {
        GMSpiderArgs.fName = "detailContent";
        GMSpiderArgs.fArgs = [];
    }

    function getBestM3u8() {
        let url = "";

        // 优先从 Nuxt / 全局变量提取
        try {
            if (unsafeWindow.__NUXT__?.state?.video?.sources?.[0]?.url) {
                url = unsafeWindow.__NUXT__.state.video.sources[0].url;
            }
        } catch (e) {}

        // 备用：从 script 中正则匹配
        if (!url) {
            $("script").each(function () {
                const text = $(this).text();
                const match = text.match(/(https?:\/\/[^\s"']+\.m3u8[^\s"']*)/i);
                if (match && match[1].includes("master") || match[1].length > 60) {
                    url = match[1];
                    return false;
                }
            });
        }

        return url;
    }

    const GmSpider = {
        homeContent: function () {
            let result = {
                class: [
                    {type_id: "new", type_name: "最新更新"},
                    {type_id: "release", type_name: "新片发布"},
                    {type_id: "uncensored-leak", type_name: "无码流出"},
                    {type_id: "chinese-subtitle", type_name: "中文字幕"},
                    {type_id: "madou", type_name: "麻豆传媒"},
                    {type_id: "actresses/ranking", type_name: "热门女优"},
                    {type_id: "genres", type_name: "所有类型"},
                ],
                list: []
            };

            // 提取首页视频
            $(".thumbnail, .video-card, a[href*='/en/']").each(function () {
                const $a = $(this).closest("a");
                const href = $a.attr("href") || $(this).attr("href");
                if (!href) return;

                result.list.push({
                    vod_id: href,
                    vod_name: $a.find("img").attr("alt") || $a.text().trim().substring(0, 60),
                    vod_pic: $a.find("img").attr("src") || $a.find("img").data("src"),
                    vod_remarks: $(this).find(".duration, .time, .text-xs").text().trim()
                });
            });

            return result;
        },

        categoryContent: function (tid, pg) {
            let result = { list: [], page: parseInt(pg), limit: 24, pagecount: 999 };
            return result;   // 列表页由 App 自动加载，核心在 detail
        },

        detailContent: function (ids) {
            const m3u8 = getBestM3u8();

            const vod = {
                vod_id: ids[0],
                vod_name: document.title.replace(/- MissAV.*/i, "").trim() || "MissAV视频",
                vod_pic: $("meta[property='og:image']").attr("content") || "",
                vod_year: $("time").first().text().trim(),
                vod_remarks: m3u8 ? "✅ 可播放" : "⚠️ 请先在浏览器播放",
                vod_actor: $(".actress, .performer, [href*='actresses']").text().trim(),
                vod_content: $("meta[name='description']").attr("content") || "",
                vod_play_from: "MissAV",
                vod_play_url: m3u8 ? `播放$${m3u8}` : "暂无地址$"
            };

            console.log("🎥 提取到的 m3u8:", m3u8);
            return { list: [vod] };
        },

        searchContent: function (key) {
            return { list: [] };   // 搜索使用网站自带
        }
    };

    $(document).ready(function () {
        if ($("#cf-wrapper, .cf-error, .challenge").length > 0) {
            console.log("⛔ Cloudflare 防护中");
        } else {
            const result = GmSpider[GMSpiderArgs.fName](...GMSpiderArgs.fArgs);
            if (typeof GmSpiderInject !== 'undefined') {
                GmSpiderInject.SetSpiderResult(JSON.stringify(result));
            }
        }
    });
})();
