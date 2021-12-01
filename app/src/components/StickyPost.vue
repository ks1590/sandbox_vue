<template>
  <div class='content-section'>
    <div class="sticky-wrap mt-3">
      <div class="sticky" v-for="sticky in $store.state.stickyPosts.stickyPosts" :key="sticky.id">
        <div class="sticky_post">
          <div class="post-header" @click="toggleMessage(sticky.id)">
            <div class="mb-0">
              <div>
                <b-form-rating
                  id="sticky__level"
                  readonly
                  :value="sticky.level"
                  inline
                ></b-form-rating>
              </div>
            </div>
            <div class="icons">
              <b-icon-caret-down-fill v-if="!actives.includes(sticky.id)"></b-icon-caret-down-fill>
              <b-icon-caret-up-fill v-else></b-icon-caret-up-fill>
              <b-icon-list v-on:click.stop v-if="$store.getters['members/filteredSelf'].isAdmin || $store.getters['members/filteredSelf'].isDev || sticky.uid === $store.state.user.uid" @click="openEditStickyModal(sticky.id)" variant="secondary"></b-icon-list>
              <b-icon-x-circle v-on:click.stop v-if="$store.getters['members/filteredSelf'].isAdmin || $store.getters['members/filteredSelf'].isDev || sticky.uid === $store.state.user.uid" @click="openDeleteStickyModal(sticky.id)" variant="secondary"></b-icon-x-circle>
            </div>
          </div>
          <div class="post-body">
            <p style="white-space: pre-wrap;" v-html="textToHyperlinkParsing(sticky.content)" :class="!actives.includes(sticky.id) ? 'headline sticky_post__content mb-0' : 'sticky_post__content mb-0'"></p>
            <p :style="!actives.includes(sticky.id) ? 'display: none':''" class="sticky_post__content mb-0 mt-1 text-right"><br>更新日 : {{eachUpdateDate(sticky.updatedAt)}} -{{ uidToMemberName(sticky.uid) }}</p>
          </div>
        </div>
      </div>
    </div>
    <b-modal
      id="edit-sticky-modal"
      ref="edit-sticky-modal"
      title="お知らせ"
      @cancel="handleEditCancel"
      @ok="handleStickyEdit"
      no-close-on-backdrop
      no-close-on-esc
      ok-title="更新"
      cancel-title="戻る"
      size="lg"
    >
      <b-form-group label="メッセージの優先順位" label-for="edit_sticky__content__input">
        <b-form-rating
          id="edit_sticky__level__input"
          v-model="edit_sticky.level"
          inline
        ></b-form-rating>
      </b-form-group>
      <b-form-group label="お知らせメッセージ" label-for="edit_sticky__content__input">
        <b-form-textarea
          id="edit_sticky__content__input"
          v-model="edit_sticky.content"
          size="sm"
          rows="12"
        ></b-form-textarea>
      </b-form-group>
      <b-form-group label="日付まで表示（日付を設定しない場合、メッセージは無期限に表示されます）" label-for="edit_sticky__expires_at__input">
        <b-form-datepicker
          id="edit_sticky__expires_at__input"
          v-model="edit_sticky.expiresAt"
          :min="new Date(new Date().setDate(new Date().getDate() + 1))"
        ></b-form-datepicker>
      </b-form-group>
      <b-form-group label="前回の更新" label-for="edit_date">
        <div id='edit_date'>{{ updateDate() }}</div>
      </b-form-group>
    </b-modal>
    <b-modal
      id="delete-sticky-modal"
      ref="delete-sticky-modal"
      title="お知らせメッセージを削除しますか？"
      @cancel="handleDeleteCancel"
      @ok="handleStickyDelete"
      no-close-on-backdrop
      no-close-on-esc
      ok-variant="danger"
      ok-title="削除"
      cancel-title="戻る"
    >
      このお知らせメッセージを削除してもよろしいですか？
    </b-modal>
  </div>
</template>

<script>
  import moment from 'moment'
  export default {
    name: "StickyPost.vue",
    data: function() {
      return {
        edit_sticky: {
          id: null,
          createdAt: null,
          updatedAt: null,
          expiresAt: null,
          content: '',
          level: 0,
          uid: '',
        },
        delete_sticky_post: '',
        actives: []
      }
    },
    methods: {
      textToHyperlinkParsing(text) {
        let parsed = text;
        const regex = /(?:(?:https?|ftp|file):\/\/|www\.|ftp\.)(?:\([-A-Z0-9+&@#\/%=~_|$?!:,.]*\)|[-A-Z0-9+&@#\/%=~_|$?!:,.])*(?:\([-A-Z0-9+&@#\/%=~_|$?!:,.]*\)|[A-Z0-9+&@#\/%=~_|$])/igm;
        let links = parsed.match(regex);
        if (links && links.length > 0) {
          for (const link of links) {
            parsed = parsed.replace(link, "<a target='_blank' href='"+link+"'>"+link+"</a>");
          }
        }
        return parsed;
      },
      uidToMemberName(uid) {
        return this.$store.state.members.availableUsers.filter(member => member.uid === uid).map(user => user.alias && user.alias !== '' ? user.alias:user.name)[0];
      },
      updateDate() {
        return moment(this.edit_sticky.updatedAt).format("Y年M月D日 H時m分");
      },
      eachUpdateDate(updatedAt){
        let date = updatedAt.toDate();
        return moment(date).format("Y年M月D日 H時m分");
      },
      toggleMessage(id) {
        if (this.actives.includes(id)) {
          this.actives = this.actives.filter(active => active !== id);
        } else {
          this.actives.push(id);
        }
      },
      openDeleteStickyModal(postid) {
        this.$bvModal.show('delete-sticky-modal');
        this.delete_sticky_post = postid;
      },
      async handleStickyDelete(bvModalEvt) {
        try {
          bvModalEvt.preventDefault();
          await this.$store.dispatch('stickyPosts/deleteStickyPost', this.delete_sticky_post)
          this.delete_sticky_post = '';
          this.$bvModal.hide('delete-sticky-modal');
        } catch (error) {
          this.$bvToast.toast('Sticky Post Delete Error\n'+error, {
            title: 'エラー',
            toaster: 'b-toaster-top-center',
            variant: 'danger',
            autoHideDelay: 3000
          });
        }
      },
      handleDeleteCancel(bvModalEvt) {
        bvModalEvt.preventDefault();
        this.delete_sticky_post = '';
        this.$nextTick(() => {
          this.$bvModal.hide('delete-sticky-modal');
        });
      },
      resetStickyParams() {
        this.edit_sticky = {
          id: null,
          createdAt: null,
          updatedAt: null,
          expiresAt: null,
          content: '',
          level: 0,
          uid: '',
        };
      },
      openEditStickyModal(id) {
        this.resetStickyParams();
        let sticky = this.$store.state.stickyPosts.stickyPosts.filter(stickypost => stickypost.id === id)[0];
        this.edit_sticky = {
          id: id,
          createdAt: sticky.createdAt ? sticky.createdAt.toDate() : null,
          updatedAt: sticky.updatedAt ? sticky.updatedAt.toDate() : null,
          expiresAt: sticky.expiresAt ? sticky.expiresAt.toDate() : null,
          content: sticky.content,
          level: sticky.level,
          uid: sticky.uid,
        };
        this.$bvModal.show('edit-sticky-modal');
      },
      async handleStickyEdit(bvModalEvt) {
        try {
          bvModalEvt.preventDefault();
          if (this.edit_sticky.content === '') {
            this.$bvToast.toast('日付やメッセージを確認して下さい。', {
              title: '確認',
              toaster: 'b-toaster-top-center',
              variant: 'warning',
              autoHideDelay: 1500
            });
            return;
          }
          const copy = JSON.parse(JSON.stringify(this.edit_sticky));
          await this.$store.dispatch('stickyPosts/editStickyPost', {docid: copy.id, parameters: copy})
          this.resetStickyParams();
          this.$bvModal.hide('edit-sticky-modal');
        } catch (error) {
          this.$bvToast.toast('Sticky Post Edit Error\n'+error, {
            title: 'エラー',
            toaster: 'b-toaster-top-center',
            variant: 'danger',
            autoHideDelay: 3000
          });
        }
      },
      handleEditCancel(bvModalEvt) {
        bvModalEvt.preventDefault();
        this.resetStickyParams();
        this.$nextTick(() => {
          this.$bvModal.hide('edit-sticky-modal');
        });
      },
    },
    async mounted() {
      if (this.$store.state.stickyPosts.stickyPostListener === null) {
        const vm = this;
        this.$store.dispatch('stickyPosts/getStickyPosts', { vm:vm });
      }
    }
  }
</script>

<style scoped>
  .sticky-wrap {
    display: flex;
    flex-flow: row wrap;
    justify-content: flex-start;
    align-content: baseline;
    align-items: baseline;
  }

  .sticky {
    flex: 1 1 33.33%;
    max-width: 33.33%;
    font-size: 14px;
    padding: 0 8px 16px;
  }

  .sticky_post {
    display: flex;
    flex-flow: column nowrap;
    border: 1px solid #ced4da;
    border-radius: 0.25rem;
  }

  .post-header {
    display: flex;
    flex-flow: row nowrap;
    justify-content: space-between;
    align-content: baseline;
    padding: 8px;
    cursor: pointer;
    align-items: flex-start;
    border-bottom: 1px solid #ced4da;
  }

  .headline {
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 1;
    -webkit-box-orient: vertical;
    overflow: hidden;
    white-space: pre-wrap;
    line-break: anywhere;
  }

  .icons {
    display: flex;
    flex-flow: row nowrap;
    justify-content: flex-end;
    align-items: flex-start;
    padding: 6px 12px;
  }

  .icons svg {
    width: 24px;
    height: 24px;
    padding-left: 8px;
  }

  .post-body {
    display: block;
    padding: 8px;
    max-width: 100%;
    cursor: pointer;
  }

  .sticky_post__content {
    line-break: anywhere;
  }

  #edit_sticky_button {
    width: 100%;
    height: 100%;
    text-align: center;
    vertical-align: center;
  }
</style>

<style>

</style>