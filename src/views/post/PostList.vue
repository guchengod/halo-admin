<template>
  <page-view>
    <a-card :bodyStyle="{ padding: '16px' }" :bordered="false">
      <div class="table-page-search-wrapper">
        <a-form layout="inline">
          <a-row :gutter="48">
            <a-col :md="6" :sm="24">
              <a-form-item label="关键词：">
                <a-input v-model="list.params.keyword" @keyup.enter="handleQuery()" />
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="24">
              <a-form-item label="文章状态：">
                <a-select v-model="list.params.status" allowClear placeholder="请选择文章状态" @change="handleQuery()">
                  <a-select-option v-for="status in Object.keys(postStatus)" :key="status" :value="status">
                    {{ postStatus[status].text }}
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="24">
              <a-form-item label="分类目录：">
                <a-select
                  v-model="list.params.categoryId"
                  :loading="categories.loading"
                  allowClear
                  placeholder="请选择分类"
                  @change="handleQuery()"
                >
                  <a-select-option v-for="category in categories.data" :key="category.id">
                    {{ category.name }}({{ category.postCount }})
                  </a-select-option>
                </a-select>
              </a-form-item>
            </a-col>

            <a-col :md="6" :sm="24">
              <span class="table-page-search-submitButtons">
                <a-space>
                  <a-button type="primary" @click="handleQuery()">查询</a-button>
                  <a-button @click="handleResetParam()">重置</a-button>
                </a-space>
              </span>
            </a-col>
          </a-row>
        </a-form>
      </div>

      <div class="table-operator">
        <router-link :to="{ name: 'PostWrite' }">
          <a-button icon="plus" type="primary">写文章</a-button>
        </router-link>
        <a-dropdown v-show="list.params.status != null && list.params.status !== '' && !isMobile()">
          <a-menu slot="overlay">
            <a-menu-item v-if="['DRAFT', 'RECYCLE'].includes(list.params.status)" key="1">
              <a href="javascript:void(0);" @click="handleEditStatusMore(postStatus.PUBLISHED.value)">
                <span>发布</span>
              </a>
            </a-menu-item>
            <a-menu-item v-if="['PUBLISHED', 'DRAFT', 'INTIMATE'].includes(list.params.status)" key="2">
              <a href="javascript:void(0);" @click="handleEditStatusMore(postStatus.RECYCLE.value)">
                <span>移到回收站</span>
              </a>
            </a-menu-item>
            <a-menu-item v-if="['RECYCLE', 'PUBLISHED', 'INTIMATE'].includes(list.params.status)" key="3">
              <a href="javascript:void(0);" @click="handleEditStatusMore(postStatus.DRAFT.value)">
                <span>草稿</span>
              </a>
            </a-menu-item>
            <a-menu-item v-if="['RECYCLE', 'DRAFT'].includes(list.params.status)" key="4">
              <a href="javascript:void(0);" @click="handleDeleteMore">
                <span>永久删除</span>
              </a>
            </a-menu-item>
          </a-menu>
          <a-button class="ml-2">
            批量操作
            <a-icon type="down" />
          </a-button>
        </a-dropdown>
      </div>
      <div class="mt-4">
        <!-- Mobile -->
        <a-list
          v-if="isMobile()"
          :dataSource="formattedPosts"
          :loading="list.loading"
          :pagination="false"
          itemLayout="vertical"
          size="large"
        >
          <a-list-item :key="index" slot="renderItem" slot-scope="item, index">
            <template slot="actions">
              <span>
                <a-icon type="eye" />
                {{ item.visits }}
              </span>
              <span @click="handleShowPostComments(item)">
                <a-icon type="message" />
                {{ item.commentCount }}
              </span>
              <a-dropdown :trigger="['click']" placement="topLeft">
                <span>
                  <a-icon type="bars" />
                </span>
                <a-menu slot="overlay">
                  <a-menu-item v-if="['PUBLISHED', 'DRAFT', 'INTIMATE'].includes(item.status)">
                    <a href="javascript:void(0);" @click="handleEditClick(item)">编辑</a>
                  </a-menu-item>
                  <a-menu-item v-else-if="item.status === 'RECYCLE'">
                    <a-popconfirm
                      :title="'你确定要发布【' + item.title + '】文章？'"
                      cancelText="取消"
                      okText="确定"
                      @confirm="handleEditStatusClick(item.id, 'PUBLISHED')"
                    >
                      <a href="javascript:void(0);">还原</a>
                    </a-popconfirm>
                  </a-menu-item>
                  <a-menu-item v-if="['PUBLISHED', 'DRAFT', 'INTIMATE'].includes(item.status)">
                    <a-popconfirm
                      :title="'你确定要将【' + item.title + '】文章移到回收站？'"
                      cancelText="取消"
                      okText="确定"
                      @confirm="handleEditStatusClick(item.id, 'RECYCLE')"
                    >
                      <a href="javascript:void(0);">回收站</a>
                    </a-popconfirm>
                  </a-menu-item>
                  <a-menu-item v-else-if="item.status === 'RECYCLE'">
                    <a-popconfirm
                      :title="'你确定要永久删除【' + item.title + '】文章？'"
                      cancelText="取消"
                      okText="确定"
                      @confirm="handleDeleteClick(item.id)"
                    >
                      <a href="javascript:void(0);">删除</a>
                    </a-popconfirm>
                  </a-menu-item>
                  <a-menu-item>
                    <a href="javascript:void(0);" rel="noopener noreferrer" @click="handleShowPostSettings(item)">
                      设置
                    </a>
                  </a-menu-item>
                </a-menu>
              </a-dropdown>
            </template>
            <template slot="extra">
              <span>
                <a-badge :status="item.statusProperty.status" :text="item.statusProperty.text" />
              </span>
            </template>
            <a-list-item-meta>
              <template slot="description">
                {{ item.createTime | moment }}
              </template>
              <span
                slot="title"
                style="max-width: 300px;display: block;white-space: nowrap;overflow: hidden;text-overflow: ellipsis;"
              >
                <a-icon
                  v-if="item.topPriority !== 0"
                  style="margin-right: 3px;"
                  theme="twoTone"
                  twoToneColor="red"
                  type="pushpin"
                />
                <a
                  v-if="['PUBLISHED', 'INTIMATE'].includes(item.status)"
                  :href="item.fullPath"
                  class="no-underline"
                  target="_blank"
                >
                  <a-tooltip :title="'点击访问【' + item.title + '】'" placement="top">{{ item.title }}</a-tooltip>
                </a>
                <a
                  v-else-if="item.status === 'DRAFT'"
                  class="no-underline"
                  href="javascript:void(0)"
                  @click="handlePreview(item.id)"
                >
                  <a-tooltip :title="'点击预览【' + item.title + '】'" placement="topLeft">{{ item.title }}</a-tooltip>
                </a>
                <a v-else class="no-underline" disabled href="javascript:void(0);">
                  {{ item.title }}
                </a>
              </span>
            </a-list-item-meta>
            <span> {{ item.summary }}... </span>
            <br />
            <br />
            <a-tag
              v-for="(category, categoryIndex) in item.categories"
              :key="'category_' + categoryIndex"
              color="blue"
              style="margin-bottom: 8px"
              @click="handleSelectCategory(category)"
              >{{ category.name }}
            </a-tag>
            <br />
            <a-tag
              v-for="(tag, tagIndex) in item.tags"
              :key="'tag_' + tagIndex"
              :color="tag.color"
              style="margin-bottom: 8px"
              >{{ tag.name }}
            </a-tag>
          </a-list-item>
        </a-list>

        <!-- Desktop -->
        <a-table
          v-else
          :columns="columns"
          :dataSource="formattedPosts"
          :loading="list.loading"
          :pagination="false"
          :rowKey="post => post.id"
          :rowSelection="{
            selectedRowKeys: selectedRowKeys,
            onChange: onSelectionChange,
            getCheckboxProps: getCheckboxProps
          }"
          :scrollToFirstRowOnChange="true"
        >
          <span slot="postTitle" slot-scope="text, record">
            <a-icon
              v-if="record.topPriority !== 0"
              style="margin-right: 3px;"
              theme="twoTone"
              twoToneColor="red"
              type="pushpin"
            />
            <a
              v-if="['PUBLISHED', 'INTIMATE'].includes(record.status)"
              :href="record.fullPath"
              class="no-underline"
              target="_blank"
            >
              <a-tooltip :title="'点击访问【' + text + '】'" placement="top">{{ text }}</a-tooltip>
            </a>
            <a
              v-else-if="record.status === 'DRAFT'"
              class="no-underline"
              href="javascript:void(0)"
              @click="handlePreview(record.id)"
            >
              <a-tooltip :title="'点击预览【' + text + '】'" placement="topLeft">{{ text }}</a-tooltip>
            </a>
            <a v-else class="no-underline" disabled href="javascript:void(0);">
              {{ text }}
            </a>
          </span>
          <span slot="status" slot-scope="statusProperty">
            <a-badge :status="statusProperty.status" :text="statusProperty.text" />
          </span>

          <span slot="categories" slot-scope="categoriesOfPost">
            <a-tag
              v-for="(category, index) in categoriesOfPost"
              :key="index"
              color="blue"
              style="margin-bottom: 8px;cursor:pointer"
              @click="handleSelectCategory(category)"
            >
              {{ category.name }}
            </a-tag>
          </span>

          <span slot="tags" slot-scope="tags">
            <a-tag v-for="(tag, index) in tags" :key="index" :color="tag.color" style="margin-bottom: 8px">
              {{ tag.name }}
            </a-tag>
          </span>

          <span
            slot="commentCount"
            slot-scope="text, record"
            style="cursor: pointer;"
            @click="handleShowPostComments(record)"
          >
            <a-badge
              :count="record.commentCount"
              :numberStyle="{ backgroundColor: '#f38181' }"
              :overflowCount="999"
              :showZero="true"
            />
          </span>

          <span slot="visits" slot-scope="visits">
            <a-badge
              :count="visits"
              :numberStyle="{ backgroundColor: '#00e0ff' }"
              :overflowCount="9999"
              :showZero="true"
            />
          </span>

          <span slot="createTime" slot-scope="createTime">
            <a-tooltip placement="top">
              <template slot="title">
                {{ createTime | moment }}
              </template>
              {{ createTime | timeAgo }}
            </a-tooltip>
          </span>

          <span slot="action" slot-scope="text, post">
            <a
              v-if="['PUBLISHED', 'DRAFT', 'INTIMATE'].includes(post.status)"
              href="javascript:void(0);"
              @click="handleEditClick(post)"
              >编辑</a
            >
            <a-popconfirm
              v-else-if="post.status === 'RECYCLE'"
              :title="'你确定要发布【' + post.title + '】文章？'"
              cancelText="取消"
              okText="确定"
              @confirm="handleEditStatusClick(post.id, 'PUBLISHED')"
            >
              <a href="javascript:void(0);">还原</a>
            </a-popconfirm>

            <a-divider type="vertical" />

            <a-popconfirm
              v-if="['PUBLISHED', 'DRAFT', 'INTIMATE'].includes(post.status)"
              :title="'你确定要将【' + post.title + '】文章移到回收站？'"
              cancelText="取消"
              okText="确定"
              @confirm="handleEditStatusClick(post.id, 'RECYCLE')"
            >
              <a href="javascript:void(0);">回收站</a>
            </a-popconfirm>

            <a-popconfirm
              v-else-if="post.status === 'RECYCLE'"
              :title="'你确定要永久删除【' + post.title + '】文章？'"
              cancelText="取消"
              okText="确定"
              @confirm="handleDeleteClick(post.id)"
            >
              <a href="javascript:void(0);">删除</a>
            </a-popconfirm>

            <a-divider type="vertical" />

            <a href="javascript:void(0);" @click="handleShowPostSettings(post)">设置</a>
          </span>
        </a-table>
        <div class="page-wrapper">
          <a-pagination
            :current="pagination.page"
            :defaultPageSize="pagination.size"
            :pageSizeOptions="['10', '20', '50', '100']"
            :total="pagination.total"
            class="pagination"
            showLessItems
            showSizeChanger
            @change="handlePageChange"
            @showSizeChange="handlePageSizeChange"
          />
        </div>
      </div>
    </a-card>

    <PostSettingModal
      :loading="postSettingLoading"
      :post="selectedPost"
      :savedCallback="onPostSavedCallback"
      :visible.sync="postSettingVisible"
      @onClose="selectedPost = {}"
    >
      <template #extraFooter>
        <a-button :disabled="selectPreviousButtonDisabled" @click="handleSelectPrevious">
          上一篇
        </a-button>
        <a-button :disabled="selectNextButtonDisabled" @click="handleSelectNext">
          下一篇
        </a-button>
      </template>
    </PostSettingModal>

    <TargetCommentDrawer
      :id="selectedPost.id"
      :description="selectedPost.summary"
      :target="`posts`"
      :title="selectedPost.title"
      :visible="postCommentVisible"
      @close="onPostCommentsClose"
    />
  </page-view>
</template>

<script>
import { mixin, mixinDevice } from '@/mixins/mixin.js'
import { PageView } from '@/layouts'
import PostSettingModal from './components/PostSettingModal.vue'
import TargetCommentDrawer from '../comment/components/TargetCommentDrawer'
import apiClient from '@/utils/api-client'

const columns = [
  {
    title: '标题',
    dataIndex: 'title',
    width: '150px',
    ellipsis: true,
    scopedSlots: { customRender: 'postTitle' }
  },
  {
    title: '状态',
    className: 'status',
    dataIndex: 'statusProperty',
    width: '100px',
    scopedSlots: { customRender: 'status' }
  },
  {
    title: '分类',
    dataIndex: 'categories',
    scopedSlots: { customRender: 'categories' }
  },
  {
    title: '标签',
    dataIndex: 'tags',
    scopedSlots: { customRender: 'tags' }
  },
  {
    title: '评论',
    width: '70px',
    dataIndex: 'commentCount',
    scopedSlots: { customRender: 'commentCount' }
  },
  {
    title: '访问',
    width: '70px',
    dataIndex: 'visits',
    scopedSlots: { customRender: 'visits' }
  },
  {
    title: '发布时间',
    dataIndex: 'createTime',
    width: '170px',
    scopedSlots: { customRender: 'createTime' }
  },
  {
    title: '操作',
    width: '180px',
    scopedSlots: { customRender: 'action' }
  }
]
const postStatus = {
  PUBLISHED: {
    value: 'PUBLISHED',
    color: 'green',
    status: 'success',
    text: '已发布'
  },
  DRAFT: {
    value: 'DRAFT',
    color: 'yellow',
    status: 'warning',
    text: '草稿'
  },
  RECYCLE: {
    value: 'RECYCLE',
    color: 'red',
    status: 'error',
    text: '回收站'
  },
  INTIMATE: {
    value: 'INTIMATE',
    color: 'blue',
    status: 'success',
    text: '私密'
  }
}
export default {
  name: 'PostList',
  components: {
    PageView,
    PostSettingModal,
    TargetCommentDrawer
  },
  mixins: [mixin, mixinDevice],
  data() {
    return {
      postStatus,
      columns,
      list: {
        data: [],
        loading: false,
        total: 0,
        hasPrevious: false,
        hasNext: false,
        params: {
          page: 0,
          size: 10,
          keyword: undefined,
          categoryId: undefined,
          status: undefined
        }
      },

      categories: {
        data: [],
        loading: false
      },

      selectedRowKeys: [],
      postSettingVisible: false,
      postSettingLoading: false,
      postCommentVisible: false,
      selectedPost: {}
    }
  },
  computed: {
    formattedPosts() {
      return this.list.data.map(post => {
        post.statusProperty = this.postStatus[post.status]
        return post
      })
    },
    pagination() {
      return {
        page: this.list.params.page + 1,
        size: this.list.params.size,
        total: this.list.total
      }
    },
    selectPreviousButtonDisabled() {
      const index = this.list.data.findIndex(post => post.id === this.selectedPost.id)
      return index === 0 && !this.list.hasPrevious
    },
    selectNextButtonDisabled() {
      const index = this.list.data.findIndex(post => post.id === this.selectedPost.id)
      return index === this.list.data.length - 1 && !this.list.hasNext
    }
  },
  beforeMount() {
    this.handleListCategories()
  },
  beforeRouteEnter(to, from, next) {
    next(vm => {
      if (to.query.page) {
        vm.list.params.page = Number(to.query.page)
      }
      if (to.query.size) {
        vm.list.params.size = Number(to.query.size)
      }

      vm.list.params.sort = to.query.sort
      vm.list.params.keyword = to.query.keyword
      vm.list.params.categoryId = to.query.categoryId
      vm.list.params.status = to.query.status

      vm.handleListPosts()
    })
  },
  watch: {
    'list.params': {
      deep: true,
      handler: function(newVal) {
        if (newVal) {
          const params = JSON.parse(JSON.stringify(this.list.params))
          const path = this.$router.history.current.path
          this.$router.push({ path, query: params }).catch(err => err)
        }
      }
    }
  },
  methods: {
    /**
     * Fetch post data
     */
    async handleListPosts(enableLoading = true) {
      try {
        if (enableLoading) {
          this.list.loading = true
        }
        const response = await apiClient.post.list(this.list.params)

        this.list.data = response.data.content
        this.list.total = response.data.total
        this.list.hasPrevious = response.data.hasPrevious
        this.list.hasNext = response.data.hasNext
      } catch (error) {
        this.$log.error(error)
      } finally {
        this.list.loading = false
      }
    },

    /**
     * Fetch categories data
     */
    async handleListCategories() {
      try {
        this.categories.loading = true

        const response = await apiClient.category.list({ sort: [], more: true })

        this.categories.data = response.data
      } catch (error) {
        this.$log.error(error)
      } finally {
        this.categories.loading = false
      }
    },
    handleEditClick(post) {
      this.$router.push({ name: 'PostEdit', query: { postId: post.id } })
    },
    onSelectionChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
      this.$log.debug(`SelectedRowKeys: ${selectedRowKeys}`)
    },
    getCheckboxProps(post) {
      return {
        props: {
          disabled: this.list.params.status == null || this.list.params.status === '',
          name: post.title
        }
      }
    },

    /**
     * Handle page change
     */
    handlePageChange(page = 1) {
      this.list.params.page = page - 1
      this.handleListPosts()
    },

    /**
     * Handle page size change
     */
    handlePageSizeChange(current, size) {
      this.$log.debug(`Current: ${current}, PageSize: ${size}`)
      this.list.params.page = 0
      this.list.params.size = size
      this.handleListPosts()
    },

    /**
     * Reset query params
     */
    handleResetParam() {
      this.list.params.keyword = undefined
      this.list.params.categoryId = undefined
      this.list.params.status = undefined
      this.handleClearRowKeys()
      this.handlePageChange(1)
      this.handleListCategories()
    },
    handleQuery() {
      this.handleClearRowKeys()
      this.handlePageChange(1)
    },
    handleSelectCategory(category) {
      this.list.params.categoryId = category.id
      this.handleQuery()
    },
    handleEditStatusClick(postId, status) {
      apiClient.post
        .updateStatusById(postId, status)
        .then(() => {
          this.$message.success('操作成功！')
        })
        .finally(() => {
          this.handleListPosts()
        })
    },
    handleDeleteClick(postId) {
      apiClient.post
        .delete(postId)
        .then(() => {
          this.$message.success('删除成功！')
        })
        .finally(() => {
          this.handleListPosts()
        })
    },
    handleEditStatusMore(status) {
      if (this.selectedRowKeys.length <= 0) {
        this.$message.info('请至少选择一项！')
        return
      }
      apiClient.post
        .updateStatusInBatch(this.selectedRowKeys, status)
        .then(() => {
          this.$log.debug(`postId: ${this.selectedRowKeys}, status: ${status}`)
          this.selectedRowKeys = []
        })
        .finally(() => {
          this.handleListPosts()
        })
    },
    handleDeleteMore() {
      if (this.selectedRowKeys.length <= 0) {
        this.$message.info('请至少选择一项！')
        return
      }
      apiClient.post
        .deleteInBatch(this.selectedRowKeys)
        .then(() => {
          this.$log.debug(`delete: ${this.selectedRowKeys}`)
          this.selectedRowKeys = []
        })
        .finally(() => {
          this.handleListPosts()
        })
    },
    handleShowPostSettings(post) {
      this.postSettingVisible = true
      this.postSettingLoading = true
      apiClient.post
        .get(post.id)
        .then(response => {
          this.selectedPost = response.data
        })
        .finally(() => {
          this.postSettingLoading = false
        })
    },
    handleShowPostComments(post) {
      apiClient.post.get(post.id).then(response => {
        this.selectedPost = response.data
        this.postCommentVisible = true
      })
    },
    handlePreview(postId) {
      apiClient.post.getPreviewLinkById(postId).then(response => {
        window.open(response.data, '_blank')
      })
    },
    handleClearRowKeys() {
      this.selectedRowKeys = []
    },
    onPostSavedCallback() {
      this.handleListPosts(false)
    },
    onPostCommentsClose() {
      this.postCommentVisible = false
      this.selectedPost = {}
      setTimeout(() => {
        this.handleListPosts(false)
      }, 500)
    },

    /**
     * Select previous post
     */
    async handleSelectPrevious() {
      const index = this.list.data.findIndex(post => post.id === this.selectedPost.id)
      if (index > 0) {
        this.postSettingLoading = true
        const response = await apiClient.post.get(this.list.data[index - 1].id)
        this.selectedPost = response.data
        this.postSettingLoading = false
        return
      }
      if (index === 0 && this.list.hasPrevious) {
        this.list.params.page--
        await this.handleListPosts()
        this.postSettingLoading = true
        const response = await apiClient.post.get(this.list.data[this.list.data.length - 1].id)
        this.selectedPost = response.data
        this.postSettingLoading = false
      }
    },

    /**
     * Select next post
     */
    async handleSelectNext() {
      const index = this.list.data.findIndex(post => post.id === this.selectedPost.id)
      if (index < this.list.data.length - 1) {
        this.postSettingLoading = true
        const response = await apiClient.post.get(this.list.data[index + 1].id)
        this.selectedPost = response.data
        this.postSettingLoading = false
        return
      }
      if (index === this.list.data.length - 1 && this.list.hasNext) {
        this.list.params.page++
        await this.handleListPosts()

        this.postSettingLoading = true
        const response = await apiClient.post.get(this.list.data[0].id)
        this.selectedPost = response.data
        this.postSettingLoading = false
      }
    }
  }
}
</script>
