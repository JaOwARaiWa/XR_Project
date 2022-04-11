<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal" role="dialog" aria-labelledby="modalTitle" aria-describedby="modalDescription">
        <header class="modal-header" id="modalTitle" >
          <slot name="header">
            วิธีการเล่นเกม ทายอารมณ์จากสีหน้า
          </slot>

          <button type="button" class="btn-close" v-on:click="closeModal" aria-label="Close modal">
            x
          </button>
        </header>

        <section class="modal-body1" id="modalDescription">
          <slot name="body">
            <div class="msg">ขั้นแรก "มาทำความรู้จักกับอารมณ์ และสีหน้าต่าง ๆ กันก่อน"</div><br />
            <div class="first">
              <span>
                1. เลือกอารมณ์ที่ต้องการจากตัวเลือกด้านข้าง<br />
                2. เปิดกล้องขึ้นมาเพื่อทำการตรวจจับอารมณ์<br />
                3. แสดงสีหน้าที่บ่งบอกถึงอารมณ์ที่เลือก<br />
                4. กดปุ่ม "ถ่าย แชะ!" เพื่อเป็นการเก็บสีหน้าที่บ่งบอกอารมณ์นั้น ๆ<br /><br />
              </span>
            </div>
            <span>
              เสร็จเรียบร้อย เท่านี้ก็จะได้ข้อมูลของสีหน้ามาแล้ว<br />
              ใน 1 อารมณ์สามารถเก็บสีหน้าได้เรื่อย ๆ เพื่อเพิ่มความแม่นยำในการตรวจจับ<br />
              <br />
            </span>
          </slot>

        </section>
        <section class="modal-body2" id="modalDescription">
          <slot name="body">
            <div class="msg">ขั้นที่ 2 "มาเริ่มเล่นเกม ทายอารมณ์จากสีหน้า กันเถอะ"</div>
              เลือกเมนู "ลองเล่น" จากตัวเลือกด้านบน การเล่นจะแบ่งเป็น 2 ฝั่ง<br /><br />
            <div>
              <div class="msg">ฝั่งแรก "ผู้แสดงสีหน้า"</div>
              <div class="second">
                1. แสดงสีหน้าตามอารมณ์ที่<br />
                2. กดปุ่ม "ตรวจจับอารมณ์" เพื่อให้ผู้ทายได้ทำการทายสีหน้า<br /><br />
              </div>
            
              <div class="msg">ฝั่งที่ 2 "ผู้ทายสีหน้า"</div>
                หลังจากที่ตรวจจับอารมณ์สำเร็จ ก็สามารถเลือกทายได้เลย<br />
                โดยถ้าทายถูกจะได้ 1 คะแนน แต่ถ้าทายผิดจะสามารถทายใหม่ได้
            </div>
          </slot>
        </section>
      </div>
    </div>
  </transition>
</template>

<script>
  export default {
    name: 'Tutorial',
    methods: {
      closeModal() {
        this.$root.$emit('closeModal');
      },
    },
  };
</script>

<style>
  .modal-backdrop {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.3);
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .modal {
    background: #FFFFFF;
    box-shadow: 2px 2px 20px 1px;
    overflow-y: auto;
    /* display: flex;
    flex-direction: column; */
    width: 50%;
    height: 90%;
  }

  .modal-header,
  .modal-footer {
    padding: 10px;
    /* display: flex; */
  }

  .modal-header {
    position: relative;
    border-bottom: 1px solid #eeeeee;
    color: #000000;
    justify-content: space-between;
    font-size: 1.8em;
    text-align: center;
  }

  .modal-footer {
    border-top: 1px solid #eeeeee;
    flex-direction: column;
  }

  .modal-body1 {
    position: relative;
    padding-top: 15px;
    height: 40%;
    font-size: 1.15em;
  }

  .first {
    text-align: left;
    margin: auto;
    width: 65%;
  }

  .modal-body2 {
    position: relative;
    padding-top: 20px;
    padding-bottom: 20px;
    height: 42.24%;
    background-color: #646490;
    color: #FFFFFF;
    font-size: 1.15em;
  }

  .second {
    text-align: left;
    margin: auto;
    width: 65%;
  }

  .btn-close {
    position: absolute;
    top: 0;
    right: 0;
    border: none;
    font-size: 30px;
    padding-right: 30px;
    padding-top: 5px;
    cursor: pointer;
    color: #000000;
    background: transparent;
  }

  .btn-green {
    color: white;
    background: #4AAE9B;
    border: 1px solid #4AAE9B;
    border-radius: 2px;
  }

  .modal-fade-enter,
  .modal-fade-leave-to {
    opacity: 0;
  }

  .modal-fade-enter-active,
  .modal-fade-leave-active {
    transition: opacity .5s ease;
  }
</style>