<template>
  <div>
    <BasicTable @register="registerTable" :searchInfo="searchInfo">
      <template #tableTitle>
        <a-button type="primary" @click="handlerReadAllMsg">全部标注已读</a-button>
      </template>
      <template #action="{ record }">
        <TableAction :actions="getActions(record)" />
      </template>
    </BasicTable>
    <DetailModal @register="register" />
  </div>
</template>
<script lang="ts" name="monitor-mynews" setup>
  import { ref, onMounted } from 'vue';
  import { BasicTable, useTable, TableAction } from '/@/components/Table';
  import DetailModal from './DetailModal.vue';
  import { getMyNewsList, editCementSend, syncNotic, readAllMsg, getOne } from './mynews.api';
  import { columns, searchFormSchema } from './mynews.data';
  import { useMessage } from '/@/hooks/web/useMessage';
  import { getToken } from '/@/utils/auth';
  import { useModal } from '/@/components/Modal';
  import { useGlobSetting } from '/@/hooks/setting';
  const glob = useGlobSetting();
  const { createMessage } = useMessage();
  const checkedKeys = ref<Array<string | number>>([]);
  const content = ref({});
  const searchInfo = { logType: '1' };
  const [register, { openModal: openDetail }] = useModal();
  import { useListPage } from '/@/hooks/system/useListPage';
  import { getLogList } from '/@/views/monitor/log/log.api';
  import { useRouter } from 'vue-router';
  import { useAppStore } from '/@/store/modules/app';
  import { useMessageHref } from '/@/views/system/message/components/useSysMessage';
  const appStore = useAppStore();
  const router = useRouter();
  const { currentRoute } = useRouter();
  const { goPage } = useMessageHref();
  // update-begin--author:liaozhiyang---date:20250709---for：【QQYUN-13058】我的消息区分类型且支持根据url参数查询类型
  const querystring = currentRoute.value.query;
  const findItem: any = searchFormSchema.find((item: any) => item.field === 'msgCategory');
  if (findItem) {
    if (querystring?.msgCategory) {
      findItem.componentProps.defaultValue = querystring.msgCategory
    } else {
      findItem.componentProps.defaultValue = null
    }
  }
  // update-end--author:liaozhiyang---date:20250709---for：【QQYUN-13058】我的消息区分类型且支持根据url参数查询类型
  const { prefixCls, tableContext } = useListPage({
    designScope: 'mynews-list',
    tableProps: {
      title: '我的消息',
      api: getMyNewsList,
      columns: columns,
      formConfig: {
        schemas: searchFormSchema,
        //update-begin---author:wangshuai---date:2024-06-11---for:【TV360X-545】我的消息列表不能通过时间范围查询---
        fieldMapToTime: [['sendTime', ['sendTimeBegin', 'sendTimeEnd'], 'YYYY-MM-DD']],
        //update-end---author:wangshuai---date:2024-06-11---for:【TV360X-545】我的消息列表不能通过时间范围查询---
      },
      beforeFetch: (params) => {
        // update-begin--author:liaozhiyang---date:20250709---for：【QQYUN-13058】我的消息区分类型且支持根据url参数查询类型
        if (querystring?.msgCategory) {
          params.msgCategory = querystring.msgCategory;
        }
        return params;
        // update-end--author:liaozhiyang---date:20250709---for：【QQYUN-13058】我的消息区分类型且支持根据url参数查询类型
      },
    },
  });
  const [registerTable, { reload }] = tableContext;
  /**
   * 操作列定义
   * @param record
   */
  function getActions(record) {
    return [
      {
        label: '查看',
        onClick: handleDetail.bind(null, record),
      },
    ];
  }

  /**
   * 查看
   */
  function handleDetail(record) {
    let anntId = record.anntId;
    editCementSend({ anntId: anntId }).then((res) => {
      reload();
      syncNotic({ anntId: anntId });
    });
    const openModalFun = ()=>{
      openDetail(true, {
        record,
        isUpdate: true,
      });
    }
    goPage(record, openModalFun);
   
  }
  // 日志类型
  function callback(key) {
    searchInfo.logType = key;
    reload();
  }

  //全部标记已读
  function handlerReadAllMsg() {
    readAllMsg({}, reload);
  }

  /**
   * 选择事件
   */
  function onSelectChange(selectedRowKeys: (string | number)[]) {
    checkedKeys.value = selectedRowKeys;
  }
  
  //update-begin-author:taoyan date:2022-8-23 for: 消息跳转，打开详情表单
  onMounted(()=>{
    initHrefModal();
  });
  function initHrefModal(){
    let params = appStore.getMessageHrefParams;
    if(params){
      let anntId = params.id;
      if(anntId){
        editCementSend({ anntId: anntId }).then(() => {
          reload();
          syncNotic({ anntId: anntId });
        });
      }
      let detailId = params.detailId;
      if(detailId){
        getOne(detailId).then(data=>{
          console.log('getOne', data)
          openDetail(true, {
            record: data,
            isUpdate: true,
          });
          appStore.setMessageHrefParams('')
        })
      }
    }
  }
  //update-end-author:taoyan date:2022-8-23 for: 消息跳转，打开详情表单

  
</script>
