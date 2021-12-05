<template>
  <Layout>
    <Tabs class-prefix="type" :data-source="recordTypeList" :value.sync="type"></Tabs>
    <ol v-if="groupedList.length>0">
      <li v-for="(group,index) in groupedList" :key="index">
        <h3 class="title">{{ beautify(group.title) }} <span>￥{{ group.total }}</span></h3>
        <ol>
          <li v-for="item in group.items" :key="item.id" class="record">
            <span>{{ tagString(item.tags) }}</span>
            <span class="notes" :style="{marginRight:'auto'}">{{ item.notes }}</span>
            <span>￥{{ item.amount }}</span>
          </li>
        </ol>
      </li>
    </ol>
    <div v-else class="noResult">
      目前没有相关记录
    </div>
  </Layout>
</template>

<style scoped lang="scss">
.noResult {
  padding: 16px;
  text-align: center;
}

%item {
  padding: 8px 16px;
  line-height: 24px;
  display: flex;
  justify-content: space-between;
  align-content: center;
}

.title {
  @extend %item
}

.item {
  @extend %item;
}

.record {
  background: white;
  @extend %item;
}

.notes {
  margin-right: auto;
  margin-left: 16px;
  color: #999;
}

</style>

<script lang="ts">
import Vue from 'vue';
import {Component} from 'vue-property-decorator';
import Tabs from '@/components/Tabs.vue';
import intervalList from '@/constants/intervalList';
import recordTypeList from '@/constants/recordTypeList';
import dayjs from 'dayjs';
import clone from '@/lib/clone';

@Component({
  components: {Tabs},
})
export default class Statistics extends Vue {
  // eslint-disable-next-line no-undef
  tagString(tags: Tag[]) {
    return tags.length === 0 ? '无' : tags.map(t => t.name).join('，');
  }

  beautify(string: string) {
    const day = dayjs(string);
    const now = dayjs();
    if (day.isSame(now, 'day')) {
      return '今天';
    } else if (day.isSame(now.subtract(1, 'day'), 'day')) {
      return '昨天';
    } else if (day.isSame(now.subtract(2, 'day'), 'day')) {
      return '前天';
    } else if (day.isSame((now), 'year')) {
      return day.format('MM月DD日');
    } else {
      return day.format('YYYY年MM月DD日');
    }
  }

  get recordList() {
    return this.$store.state.recordList;
  }

  get groupedList() {
    const {recordList} = this;

    // eslint-disable-next-line no-undef
    const newList = clone(recordList).filter((r: RecordItem) => r.type === this.type).sort((a: RecordItem, b: RecordItem) => dayjs(b.createdAt).valueOf() - dayjs(a.createdAt).valueOf());
    if (newList.length === 0) {return [] as Result;}
    // eslint-disable-next-line no-undef
    type Result = { title: string, total?: number, items: RecordItem[] }[]
    const result: Result = [{title: dayjs(newList[0].createdAt).format('YYYY-MM-DD'), items: [newList[0]]}];
    for (let i = 1; i < newList.length; i++) {
      const current = newList[i];
      const last = result[result.length - 1];
      if (dayjs(last.title).isSame(dayjs(current.createdAt), 'day')) {
        last.items.push(current);
      } else {
        result.push({title: dayjs(current.createdAt).format('YYYY-MM-DD'), items: [current]});
      }
    }
    result.map(group => {
      group.total = group.items.reduce((sum, item) => sum + item.amount, 0);
    });
    return result;
  }

  beforeCreated() {
    this.$store.commit('fetchRecords');
  }

  type = '-';
  recordTypeList = recordTypeList;
}
</script>

<style lang="scss" scoped>
::v-deep .type-tabs-item {
  background-color: #c4c4c4;

  &.selected {

    background-color: white;

    &::after {
      display: none;
    }
  }
}

::v-deep .interval-tabs-item {
  height: 48px;
}
</style>