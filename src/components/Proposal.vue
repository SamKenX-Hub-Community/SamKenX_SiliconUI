<script setup lang="ts">
import { ref } from 'vue';
import { _rt, shortenAddress } from '@/helpers/utils';
import { useActions } from '@/composables/useActions';
import type { Proposal as ProposalType, Choice } from '@/types';

const props = defineProps<{ proposal: ProposalType }>();

const { vote } = useActions();
const modalOpenVotes = ref(false);
const modalOpenTimeline = ref(false);
const sendingType = ref<null | number>(null);

async function handleVoteClick(choice: Choice) {
  sendingType.value = choice;

  try {
    await vote(props.proposal, choice);
  } finally {
    sendingType.value = null;
  }
}
</script>
<template>
  <div>
    <div class="border-b mx-4 py-3 flex">
      <div class="flex-auto">
        <router-link
          :to="{
            name: 'proposal',
            params: {
              pid: proposal.proposal_id,
              id: `${proposal.network}:${proposal.space.id}`
            }
          }"
          class="block"
        >
          <h3 v-text="proposal.title || `Proposal #${proposal.proposal_id}`" />
        </router-link>
        <router-link
          :to="{
            name: 'user',
            params: { id: `${proposal.network}:${proposal.author.id}` }
          }"
        >
          <Stamp :id="proposal.author.id" :size="24" class="mr-1" />
          {{ shortenAddress(proposal.author.id) }}
        </router-link>
        <span>
          ·
          <a
            class="text-skin-text"
            @click="modalOpenTimeline = true"
            v-text="_rt(proposal.created)"
          />
        </span>
      </div>
      <div class="hidden md:block">
        <Vote :proposal="proposal">
          <template #unsupported><div /></template>
          <template #waiting><div /></template>
          <template #voted>
            <Results :proposal="proposal" />
          </template>
          <template #ended>
            <Results :proposal="proposal" />
          </template>
          <div class="space-x-2 py-2">
            <UiButton
              class="w-full !text-green !border-green !w-[40px] !h-[40px] !px-0"
              :loading="sendingType === 1"
              @click="handleVoteClick(1)"
            >
              <IH-check class="inline-block" />
            </UiButton>
            <UiButton
              class="w-full !text-red !border-red !w-[40px] !h-[40px] !px-0"
              :loading="sendingType === 2"
              @click="handleVoteClick(2)"
            >
              <IH-x class="inline-block" />
            </UiButton>
            <UiButton
              class="w-full !text-gray-500 !border-gray-500 !w-[40px] !h-[40px] !px-0"
              :loading="sendingType === 3"
              @click="handleVoteClick(3)"
            >
              <IH-arrow-sm-right class="inline-block" />
            </UiButton>
          </div>
        </Vote>
      </div>
    </div>
    <teleport to="#modal">
      <ModalVotes :open="modalOpenVotes" :proposal="proposal" @close="modalOpenVotes = false" />
      <ModalTimeline
        :open="modalOpenTimeline"
        :proposal="proposal"
        @close="modalOpenTimeline = false"
      />
    </teleport>
  </div>
</template>
