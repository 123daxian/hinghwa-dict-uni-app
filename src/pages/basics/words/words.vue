<template>
  <view>
    <!--导航栏-->
    <cu-custom />

    <!--贡献语音的按钮-->
    <button
      v-show="tabIndex===tabs.indexOf('发音')"
      class="cu-btn icon lg bg-blue shadow contributing-button"
      @tap="toUploadPronunciationPage(id)"
    >
      <text class="cuIcon-voice" />
    </button>

    <!--标题栏：词语-->
    <view class="flex padding solid-bottom">
      <view class="flex-sub">
        <view class="text-bold text-sl">
          {{ word.word }}
        </view>
      </view>
      <view class="flex-sub">
        <view class="text-grey text-df">
          访问：{{ word.views }}
        </view>
        <view class="text-grey text-df margin-top-xs">
          来源：
          <text
            :data-id="word.contributor.id"
            class="text-blue"
            @tap="toUserPage(word.contributor.id)"
          >
            {{ word.contributor.nickname }}
          </text>
        </view>
      </view>
    </view>

    <!--标题栏：发音-->
    <view class="padding solid-bottom">
      <view>
        <text
          class="text-bold text-xl"
          space="ensp"
        >
          {{ word.standard_pinyin }}
        </text>
        <WordPronunciationButton
          v-if="!word.source"
          :pinyin="word.standard_pinyin"
        />
      </view>
      <view>
        <text
          class="text-grey text-xl"
          space="ensp"
        >
          /{{ word.standard_ipa }}/
        </text>
        <WordPronunciationButton
          :ipa="word.standard_ipa"
          :source="word.source"
        />
      </view>
    </view>

    <!--标签选择-->
    <scroll-view
      class="bg-white nav text-center"
      scroll-x
    >
      <view
        v-for="(item, index) in tabs"
        :key="index"
        :class="'cu-item ' + (index === tabIndex ? 'text-blue cur' : '')"
        @tap="tabIndex=index"
      >
        {{ item }}
      </view>
    </scroll-view>

    <!--标签内容-->
    <swiper
      :current="tabIndex"
      class="screen-swiper"
      duration="500"
      style="height: 150vh"
      @change="tabSlide"
    >
      <!--释义-->
      <swiper-item class="margin">
        <view
          v-for="(item, index) in definition"
          :key="index"
          class="margin-top-sm solid-bottom"
          style="width: 92vw"
        >
          <view class="text-bold text-xl">
            {{ item.content }}
          </view>
          <view
            v-for="(jtem, index1) in item.example"
            :key="index1"
            class="padding-sm"
          >
            <button
              class="cu-btn bg-red"
              style="width: 8vw"
            >
              {{ jtem.type }}
            </button>
            <text>{{ jtem.content }}</text>
            <text class="text-grey">
              {{ jtem.explain }}
            </text>
          </view>
        </view>

        <!--相关词汇-->
        <view
          v-if="word.related_words.length !== 0"
          class="margin-top-xl"
        >
          <view class="text-bold text-xl">
            相关词汇
          </view>
          <view class="margin-top-sm">
            <text
              v-for="(item, index) in word.related_words"
              :key="index"
              class="text-blue"
              @tap="toWordPage(item.id)"
            >
              {{ item.word + " " }}
            </text>
          </view>
        </view>
      </swiper-item>

      <!--发音-->
      <swiper-item>
        <view
          v-if="pronunciation.length === 0"
          class="margin"
        >
          <text class="text-grey">
            该词语暂无语音哦～点击右下方贡献语音～
          </text>
        </view>
        <view
          v-for="(item, index) in pronunciation"
          :key="index"
          class="padding solid-bottom"
        >
          <view class="flex justify-between align-center">
            <view>
              <view>
                <text class="text-bold">
                  {{ item.pronunciation.pinyin }}
                </text>
                <text class="text-grey">
                  {{ ` /${item.pronunciation.ipa}/ ` }}
                </text>
                <text
                  class="cuIcon-notificationfill text-blue"
                  @tap="playAudio(item.pronunciation.source)"
                />
              </view>
              <view class="margin-top">
                <text>来源：</text>
                <text
                  class="text-blue"
                  @tap="toUserPage(item.contributor.id)"
                >
                  {{ item.contributor.nickname }}
                </text>
                <text>
                  {{ ` （${item.pronunciation.county} ${item.pronunciation.town}）` }}
                </text>
              </view>
            </view>
            <view class="text-xl">
              <text class="cuIcon-like" />
            </view>
          </view>
        </view>
      </swiper-item>

      <!--其他-->
      <swiper-item class="margin">
        <!--空白占位-->
        <text class="text-gray text-sm">
          内容有误？想要更新？请反馈给管理员或在网页端发起修改~
        </text>

        <!--附注-->
        <view
          v-if="word.annotation.length !== 0"
          class="margin-right"
        >
          <view class="text-bold text-xl">
            附注
          </view>
          <text class="margin-top-sm text-grey">
            {{ word.annotation }}
          </text>
        </view>

        <!--普通话词汇-->
        <view
          v-if="word.mandarin.length !== 0"
          class="margin-top"
        >
          <view class="text-bold text-xl">
            普通话词汇
          </view>
          <view class="margin-top-sm">
            <text
              v-for="(item, index) in word.mandarin"
              :key="index"
            >
              {{ item + " " }}
            </text>
          </view>
        </view>
        <!--TODO: 相关文章-->
      </swiper-item>
    </swiper>
  </view>
</template>

<script>
import { getWordDetails } from '@/services/word';
import {
  toIndexPage, toUserPage,
} from '@/routers';
import { playAudio } from '@/utils/audio';
import { getPronunciations } from '@/services/pronunciation';
import WordPronunciationButton from '@/components/WordPronunciationButton';
import { toUploadPronunciationPage, toWordPage } from '@/routers/word';

const app = getApp();
export default {
  components: { WordPronunciationButton },
  data() {
    return {
      playAudio,
      toUserPage,
      toWordPage,
      toUploadPronunciationPage,
      id: 0,
      word: {
        id: 0,
        word: '',
        definition: '',
        definitions: [],
        contributor: {
          nickname: '',
          avatar: '',
          id: 0,
        },
        annotation: '',
        mandarin: [],
        related_words: [],
        related_articles: [],
        views: 0,
        standard_ipa: '',
        standard_pinyin: '',
        source: '',
      },
      definition: [],
      pronunciation: [],
      tabIndex: 0, // 当前选中的标签页
      tabs: ['释义', '发音', '其他'], // 标签页标题
      isShare: false, // 是否是直接分享进入的
    };
  },

  /**
   * 右上角分享事件
   */
  onShareAppMessage() {
    let title = `${this.word.word}：`;
    this.word.definitions.forEach((item) => {
      title += `${item.content}；`;
    });
    return {
      title,
      path: `/pages/basics/words/words?id=${this.id}&share=1`,
      success: () => {
        uni.showToast({
          title: '分享成功',
          icon: 'success',
          duration: 2000,
        });
      },
      fail: () => {
        uni.showToast({
          title: '分享失败',
          icon: 'none',
          duration: 2000,
        });
      },
    };
  },

  /**
   * 生命周期函数--监听页面加载
   * @param options
   */
  async onLoad(options) {
    this.isShare = !!options.share;
    this.id = options.id;
    this.word = await getWordDetails(options.id);
    this.definition = this.word.definitions;
    this.pronunciation = await getPronunciations({ word: options.id });
  },
  methods: {
    /**
     * 切换标签页
     */
    tabSlide(e) {
      this.tabIndex = e.detail.current;
    },
  },
};
</script>
<style>
page {
  background-color: #ffffff;
}

.contributing-button {
  position: fixed;
  bottom: 10vh;
  right: 6vw;
  z-index: 1024;
}
</style>
