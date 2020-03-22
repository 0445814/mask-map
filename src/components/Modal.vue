<template>
  <!-- Modal -->
  <div class="modal fade" id="modal" tabindex="-1" role="dialog" data-backdrop="static">
    <div class="modal-md modal-dialog modal-dialog-centered" role="document">
      <div class="modal-content">
        <div class="modal-header text-white">
          <slot name="modal-title" />

          <button type="button" class="close" data-dismiss="modal" aria-label="Close">
            <span class="text-white" aria-hidden="true">&times;</span>
          </button>
        </div>
        <div class="modal-body d-flex justify-content-lg-between align-items-center">
          <slot name="modal-content" />
          <slot name="modal-content-img" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from 'jquery';

export default {
  mounted() {
    $('#modal').modal({
      show: true,
    });
  },
};
</script>

<style lang="scss" scoped>
.modal {
  z-index: 9999;
  .modal-header {
    background: #73c0d8;
  }
}
</style>
