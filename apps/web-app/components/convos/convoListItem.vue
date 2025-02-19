<script setup lang="ts">
  import { useTimeAgo } from '@vueuse/core';
  import type {
    ConvoParticipantEntry,
    UserConvosDataType
  } from '~/composables/types';

  type Props = {
    convo: UserConvosDataType[number];
  };
  const props = defineProps<Props>();

  const timeAgo = useTimeAgo(props.convo.lastUpdatedAt || new Date());

  const participantArray = ref<ConvoParticipantEntry[]>([]);
  const author = ref<ConvoParticipantEntry>();
  const firstEntryAuthor =
    props.convo.entries[0].author || props.convo.participants[0];
  const authorEntryPublicId = computed(() => {
    return (
      firstEntryAuthor.orgMember?.publicId ||
      firstEntryAuthor.userGroup?.publicId ||
      firstEntryAuthor.contact?.publicId
    );
  });
  const authorEntryName = computed(() => {
    return (
      firstEntryAuthor.userGroup?.publicId ||
      firstEntryAuthor.contact?.publicId ||
      firstEntryAuthor.orgMember?.profile.firstName +
        ' ' +
        firstEntryAuthor.orgMember?.profile.lastName ||
      'Participant'
    );
  });

  for (const participant of props.convo.participants) {
    const {
      participantPublicId,
      participantTypePublicId,
      avatarPublicId,
      participantName,
      participantType,
      participantColor,
      participantRole
    } = useUtils().convos.useParticipantData(participant);
    const participantData: ConvoParticipantEntry = {
      participantPublicId: participantPublicId,
      typePublicId: participantTypePublicId,
      avatarPublicId: avatarPublicId,
      name: participantName,
      type: participantType,
      role: participantRole,
      color: participantColor
    };
    if (participantTypePublicId === authorEntryPublicId.value) {
      author.value = participantData;
    } else {
      participantArray.value.push(participantData);
    }
  }
</script>
<template>
  <button
    class="bg-gray-50 hover:bg-gray-100 dark:bg-gray-900 dark:hover:bg-gray-800 max-w-full flex flex-col justify-between gap-0 rounded-lg">
    <div class="h-fit w-full flex flex-row items-center gap-6">
      <UnUiAvatarPlus
        :avatars="participantArray"
        :primary="author"
        size="lg" />
      <div class="w-full flex flex-col gap-1 overflow-hidden">
        <!-- <div class="text-base text-left w-full overflow-hidden text-sm">
          <span class="line-clamp-2 font-bold">{{ authorName }}</span>
        </div> -->
        <div class="w-full overflow-hidden text-left text-xs">
          <span class="truncate text-xs font-italic">
            Re: {{ props.convo.subjects[0].subject }}
          </span>
        </div>
        <div class="w-full overflow-hidden text-left text-base text-sm">
          <span class="line-clamp-2">
            <span class="font-bold">{{ authorEntryName }} </span>
            : {{ props.convo.entries[0].bodyPlainText }}
          </span>
        </div>
      </div>
    </div>
    <div class="w-full flex flex-row items-center justify-end gap-1">
      <div class="min-w-fit overflow-hidden text-right text-xs text-base-11">
        <span class="">{{ timeAgo }}</span>
      </div>
    </div>
  </button>
</template>
