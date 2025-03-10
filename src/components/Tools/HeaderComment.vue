<template>
  <a-popover
    v-model="visible"
    :arrowPointAtCenter="true"
    :autoAdjustOverflow="true"
    :overlayStyle="{ width: '300px', top: '50px' }"
    placement="bottomRight"
    title="待审核评论"
    trigger="click"
  >
    <template slot="content">
      <div class="custom-tab-wrapper">
        <a-tabs v-model="activeKey" :animated="{ inkBar: true, tabPane: false }" @change="handleTabsChanged">
          <a-tab-pane key="post" tab="文章">
            <a-list :dataSource="converttedPostComments" :loading="postCommentsLoading">
              <a-list-item slot="renderItem" slot-scope="item">
                <a-list-item-meta>
                  <a-avatar slot="avatar" :src="item.avatar" class="bg-white" size="large" />
                  <template slot="title">
                    <a :href="item.authorUrl" target="_blank">{{ item.author }}</a
                    >：<span v-html="item.content"></span>
                  </template>
                  <template slot="description">
                    {{ item.createTime | timeAgo }}
                  </template>
                </a-list-item-meta>
              </a-list-item>
            </a-list>
          </a-tab-pane>
          <a-tab-pane key="sheet" tab="页面">
            <a-list :dataSource="converttedSheetComments" :loading="sheetCommentsLoading">
              <a-list-item slot="renderItem" slot-scope="item">
                <a-list-item-meta>
                  <a-avatar slot="avatar" :src="item.avatar" class="bg-white" size="large" />
                  <template slot="title">
                    <a :href="item.authorUrl" target="_blank">{{ item.author }}</a
                    >：<span v-html="item.content"></span>
                  </template>
                  <template slot="description">
                    {{ item.createTime | timeAgo }}
                  </template>
                </a-list-item-meta>
              </a-list-item>
            </a-list>
          </a-tab-pane>
        </a-tabs>
      </div>
    </template>
    <span class="header-comment">
      <a-badge v-if="postComments.length > 0 || sheetComments.length > 0" dot>
        <a-icon type="bell" />
      </a-badge>
      <a-badge v-else>
        <a-icon type="bell" />
      </a-badge>
    </span>
  </a-popover>
</template>

<script>
import apiClient from '@/utils/api-client'
import { marked } from 'marked'

export default {
  name: 'HeaderComment',
  data() {
    return {
      activeKey: 'post',
      visible: false,
      postComments: [],
      postCommentsLoading: false,
      sheetComments: [],
      sheetCommentsLoading: false
    }
  },
  computed: {
    converttedPostComments() {
      return this.postComments.map(comment => {
        comment.content = marked.parse(comment.content)
        return comment
      })
    },
    converttedSheetComments() {
      return this.sheetComments.map(comment => {
        comment.content = marked.parse(comment.content)
        return comment
      })
    }
  },
  created() {
    this.handleListPostAuditingComments(false)
    this.handleListSheetAuditingComments(false)
  },
  watch: {
    visible(value) {
      if (value) {
        if (this.activeKey === 'post') {
          this.handleListPostAuditingComments(false)
        } else if (this.activeKey === 'sheet') {
          this.handleListSheetAuditingComments(false)
        }
      }
    }
  },
  methods: {
    handleListPostAuditingComments(enableLoading = true) {
      if (enableLoading) {
        this.postCommentsLoading = true
      }
      apiClient.comment
        .latest('posts', 5, 'AUDITING')
        .then(response => {
          this.postComments = response.data
        })
        .finally(() => {
          this.postCommentsLoading = false
        })
    },
    handleListSheetAuditingComments(enableLoading = true) {
      if (enableLoading) {
        this.sheetCommentsLoading = true
      }
      apiClient.comment
        .latest('sheets', 5, 'AUDITING')
        .then(response => {
          this.sheetComments = response.data
        })
        .finally(() => {
          this.sheetCommentsLoading = false
        })
    },
    handleTabsChanged(activeKey) {
      if (activeKey === 'post') {
        this.handleListPostAuditingComments()
      } else if (activeKey === 'sheet') {
        this.handleListSheetAuditingComments()
      }
    }
  }
}
</script>
<style lang="less" scoped>
.header-comment {
  display: inline-block;
  transition: all 0.3s;

  span {
    vertical-align: initial;
  }
}
</style>
