<template>
  <el-dialog custom-class="user-list__wrapper" :visible.sync="_isVisible" width="26.427vw" title="添加新成员" center :append-to-body="appendToBody">
    <div style="text-align: -webkit-center">
      <el-input class="user__search" prefix-icon="el-icon-search" placeholder="输入用户名以搜索" v-model.trim="username"></el-input>
      <div id="user-list">
        <div class="user__item" v-for="u in curList" :key="u.id">
          <user-avatar class="user__avatar" :username="u.username" :job="u.jobId">
          </user-avatar>
          <div class="user__name">
            {{u.username}}
          </div>
          <el-button icon="el-icon-plus" type="text" class="add" @click="add(u)"></el-button>
        </div>
      </div>
      <div>
        <p class="tips">已添加的成员</p>
        <div class="added-users">
          <member-icons :list="addedUser" :showAdd="false" @remove="remove"></member-icons>
        </div>
        <el-button type="info" id="ensure" @click="ensure">确定</el-button>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import MemberIcons from './MemberIcons';
import UserAvatar from './UserAvatar';

export default {
  name: 'UserListDialog',
  components: {
    'member-icons': MemberIcons,
    'user-avatar': UserAvatar,
  },
  props: {
    isVisible: {
      type: Boolean,
      required: true,
    },
    list: {
      type: Array,
      default: [],
    },
    value: {
      type: Array,
      default: [],
    },
    appendToBody: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      username: '',
      // addedUser: this.value.slice(0),
    };
  },
  computed: {
    _isVisible: {
      get() { return this.isVisible; },
      set(newVal) { this.$emit('update:isVisible', newVal); },
    },
    curList() {
      let curList = [];
      curList = this.list.filter(user => user.username.includes(this.username));
      return curList;
    },
    addedUser: {
      get() {
        return this.value.slice(0);
      },
      set(v) {
        this.$emit('input', v);
      },
    },
  },
  methods: {
    add(user) {
      let temp = this.addedUser.slice(0);
      let ids = temp.map(u => u.id);
      ids.indexOf(user.id) === -1 && temp.push(user);
      this.addedUser = temp;
    },
    remove(id) {
      this.addedUser = this.addedUser.filter(user => user.id !== id);
    },
    ensure() {
      this.$emit('input', this.addedUser);
      this._isVisible = false;
    },
  },
};
</script>

<style lang="scss">
.user-list__wrapper {
  height: 75.9114vh;
  .el-dialog__header {
    font-size: 14px;
    border-bottom: 1px solid $border;
  }
  .el-dialog__body {
    padding: 20px;
  }
}

/*定义滚动条宽高及背景，宽高分别对应横竖滚动条的尺寸*/
#user-list::-webkit-scrollbar {
  width: 8px;
  height: 16px;
  background-color: #f5f5f5;
}
/*定义滚动条的轨道，内阴影及圆角*/
#user-list::-webkit-scrollbar-track {
  box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  background-color: #f5f5f5;
}
/*定义滑块，内阴影及圆角*/
#user-list::-webkit-scrollbar-thumb {
  height: 20px;
  border-radius: 10px;
  box-shadow: inset 0 0 6px #eee;
  background-color: #ddd;
}
</style>

<style lang="scss" scoped>
.user__search {
  margin-bottom: 36px;
  @include setSize(236px, 36px);
}
#user-list {
  height: 35.547vh;
  overflow: auto;
}
.user__item {
  @include setSize(236px, 36px);
  box-sizing: border-box;
  overflow: hidden;
  margin-bottom: 10px;
}
.user__avatar {
  margin-right: 20px;
  float: left;
}

.user__name {
  vertical-align: middle;
  font-size: 14px;
  line-height: 36px;
  color: $black;
  float: left;
}

.add {
  @include setSize(36px, 36px);
  font-size: 18px;
  font-weight: bolder;
  float: right;
  padding: 8px 0;
  color: $black;
  &:hover {
    color: $default;
  }
}
.tips {
  color: $tips;
  font-size: 12px;
  line-height: 17px;
  text-align: left;
  margin: 7px 0;
}

.added-users {
  @include setSize(100%, 5.21vh);
  text-align: left;
  white-space: nowrap;
  overflow-y: hidden;
  overflow-x: auto;
  &::-webkit-scrollbar {
    width: 8px;
    height: 4px;
    background-color: #f5f5f5;
  }
  &::-webkit-scrollbar-track {
    box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.1);
    border-radius: 10px;
    background-color: #f5f5f5;
  }
  &::-webkit-scrollbar-thumb {
    height: 20px;
    border-radius: 10px;
    box-shadow: inset 0 0 6px #eee;
    background-color: #bbb;
  }
}

#ensure {
  @include setSize(94px, 36px);
  box-sizing: border-box;
  display: block;
  font-size: 12px;
  padding: 12px 0;
  margin-top: 20px;
  background-color: $default;
  &:hover {
    opacity: 0.9;
  }
}
</style>



