<template>
  <div id="figure"
       :class="['figure', figureClassName]"
       :style="'margin-left: '+ left + '%'"
       :data-weight="weight"
  >
    <div class="figure__inwrap">
      <span>{{weight}}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: "Block",
  props: {
    figureClassName: {
      type: String
    },
    weight: {
      type: Number
    },
    left: {
      type: Number
    }
  },
}
</script>

<style scoped lang="scss">
$fallingColor: #42b983;
$landedColor: #91514e;

@mixin colorSchema($schemaColor) {
  color: $schemaColor;
  background-color: lighten($schemaColor, 45%);
  border: 1px solid $schemaColor;
}

.figure {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  margin: 24px 0 0 0;
  max-width: 70px;
  overflow: hidden;

  @include colorSchema($fallingColor);
  border-radius: 3px;

  &.__square { border-radius: 2px }

  &.__circle { border-radius: 50% }

  &.__square, &.__circle { aspect-ratio: 1/1 }

  &.__triangle {
    overflow: visible;
    width: 0;
    height: 0;
    background-color: transparent;
    border-top: transparent;
    border-left: 35px solid transparent;
    border-right: 35px solid transparent;
    border-bottom: 70px solid lighten($fallingColor, 45%);

    .figure__inwrap {
      margin-top: 85px;
      font-size: 2.33em;
    }
  }

  &__inwrap {
    flex: 1;
    font-size: 3.22em;
    font-weight: bold;
    text-align: center;
    padding-top: 4px;
  }

  &__p {
    line-height: 1.4;
    padding: 2px 4px;
    font-size: 12px;
    text-decoration: underline;
    cursor: pointer;
  }

  &__landed {
    position: absolute;
    bottom: 2px;
    width: 70px;
    margin-top: unset;

    @include colorSchema($landedColor);

    &.__triangle {
      border-bottom-color: lighten($landedColor, 45%);
      bottom: 3px;
    }
  }
}
</style>