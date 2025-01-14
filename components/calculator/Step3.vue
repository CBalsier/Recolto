<template>
  <div class="flex flex-col">
    <div class="bg-purple p-4 rounded-2xl mb-4 relative">
      <h2
        class="absolute font-semibold -top-2 -left-2 w-8 h-8 text-center rounded-full bg-purple shadow-xl border border-white"
      >
        6
      </h2>
      <h3 class="text-center">Votre estimation</h3>
    </div>

    <div v-if="result && !loading" class="flex flex-wrap">
      <div class="w-2/3 text-md lg:text-lg font-semibold mb-2 self-center">
        <div class="flex">
          <p class="w-auto md:w-5/6">Volume idéal de stockage</p>
          <UPopover
            class="hidden md:ml-2 md:w-1/6 md:block md:self-center" mode="hover"
            :ui="{
              width: 'max-w-md',
              background: 'bg-purple dark:bg-gray-900',
           }"
          >
            <UIcon name="i-heroicons-information-circle" class="w-6 h-6" />
            <template #panel>
              <p class="text-sm m-1">
                Le mot clé dans une utilisation optimale est <span
                class="underline"
              >roulement</span> et non stockage. Pour une utilisation adaptée, privilégiez un roulement régulier et un renouvellement à chaque épisode de pluie de votre récupérateur plutôt qu'un stockage trop long.
              </p>
            </template>
          </UPopover>
        </div>
      </div>
      <div class="w-1/3 text-xl md:text-2xl font-bold flex justify-end self-center">
        {{ (result.idealCapacity).toLocaleString("fr-FR") }} L
      </div>

      <div class="w-2/3 text-md lg:text-lg font-semibold mb-2 self-center">
        Économie estimée
      </div>
      <div class="w-1/3 text-xl md:text-2xl font-bold flex justify-end self-center">
        {{ (result.savingForLastKnownYear).toLocaleString("fr-FR") }} €/an
      </div>

      <hr class="border border-t border-purple w-full my-2">

      <div class="w-2/3 text-sm lg:text-base font-semibold mb-2 self-center">
        <div class="flex">
          <p class="w-auto md:w-5/6">
            Prix de l'eau {{ getDivisionForWaterPrice }} en {{
              result.waterPrice.year
            }}
          </p>
          <UPopover
            class="hidden md:ml-2 md:w-1/6 md:block md:self-center" mode="hover"
            :ui="{
            width: 'max-w-md',
            background: 'bg-purple dark:bg-gray-900',
          }"
          >
            <UIcon name="i-heroicons-information-circle" class="w-6 h-6 " />
            <template #panel>
              <p class="text-sm m-1">
                Ce tarif est fourni par l'observatoire national des services d'eau et assainissement.
                <a
                  href="https://services.eaufrance.fr/carte-interactive"
                  target="_blank"
                  rel="noopener"
                  class="underline"
                >
                  (Voir leur carte)
                </a>
              </p>
            </template>
          </UPopover>
        </div>
      </div>
      <div class="w-1/3 text-base md:text-lg font-bold flex justify-end self-center">
        {{ (result.waterPrice.price).toLocaleString("fr-FR") }} €/m³
      </div>

      <div class="w-2/3 text-sm lg:text-base font-semibold mb-2 self-center">
        Vos besoins en eau
      </div>
      <div class="w-1/3 text-base md:text-lg font-bold flex justify-end self-center">
        {{ (result.waterNeeds).toLocaleString("fr-FR") }} L/an
      </div>
    </div>
    <div v-if="!result && !loading">
      Malheureusement, il a été impossible de réaliser l'estimation, veuillez soit réessayer ou modifier certains paramètres.
    </div>
    <div v-if="loading">
      <div class="flex items-center space-x-4 mt-2">
        <USkeleton class="h-12 w-12" :ui="{ rounded: 'rounded-full' }" />
        <div class="space-y-2">
          <USkeleton class="h-4 w-[250px]" />
          <USkeleton class="h-4 w-[200px]" />
        </div>
      </div>
      <div class="flex items-center space-x-4 mt-2">
        <USkeleton class="h-12 w-12" :ui="{ rounded: 'rounded-full' }" />
        <div class="space-y-2">
          <USkeleton class="h-4 w-[250px]" />
          <USkeleton class="h-4 w-[200px]" />
        </div>
      </div>
    </div>

    <hr class="border border-t border-purple w-full my-2">

    <div v-if="result && !loading" class="w-full">
      <h3
        class="font-semibold text-center text-white"
      >
        Analyse de vos besoins au regard des précipitations enregistrées l'année :
      </h3>
      <div class="flex w-full gap-1 sm:gap-2">
        <UButton
          variant="outline"
          :trailing="false"
          @click="selectedScenario('recently')"
          class="my-3 md:my-2 bg-purple border border-white flex justify-center items-center text-white disabled:bg-purple-300 ring-purple hover:bg-purple-900 dark:bg-purple dark:text-white dark:hover:bg-purple-900 flex-1 px-0.5"
          :class="{'bg-purple-200 text-purple hover:bg-purple-200 dark:bg-purple-200 dark:text-purple dark:hover:bg-purple-200': graphToDisplay === 'recently'}"
        >
          <div class="flex flex-col text-xs md:text-base">
            <span>La plus récente</span>
            <span>{{ props.result?.lastKnownYear }}</span>
          </div>
        </UButton>
        <UButton
          variant="outline"
          :trailing="false"
          @click="selectedScenario('driest')"
          class="my-3 md:my-2 bg-purple border border-white flex justify-center items-center text-white disabled:bg-purple-300 ring-purple hover:bg-purple-900 dark:bg-purple dark:text-white dark:hover:bg-purple-900 flex-1 px-0.5"
          :class="{'bg-purple-200 text-purple hover:bg-purple-200 dark:bg-purple-200 dark:text-purple dark:hover:bg-purple-200': graphToDisplay === 'driest'}"
        >
          <div class="flex flex-col text-xs md:text-base">
            <span>La plus sèche</span>
            <span>{{ props.result?.driestYear }}</span>
          </div>
        </UButton>
        <UButton
          variant="outline"
          :trailing="false"
          @click="selectedScenario('wettest')"
          class="my-3 md:my-2 bg-purple border border-white flex justify-center items-center text-white disabled:bg-purple-300 ring-purple hover:bg-purple-900 dark:bg-purple dark:text-white dark:hover:bg-purple-900 flex-1 px-0.5"
          :class="{'bg-purple-200 text-purple hover:bg-purple-200 dark:bg-purple-200 dark:text-purple dark:hover:bg-purple-200': graphToDisplay === 'wettest'}"
        >
          <div class="flex flex-col text-xs md:text-base">
            <span>La plus pluvieuse</span>
            <span>{{ props.result?.wettestYear }}</span>
          </div>
        </UButton>
      </div>
    </div>

    <div
      id="refChart"
    />

    <div v-if="result && !loading" class="text-xs font-medium">
      <p>
        Cette estimation se base sur les données
        <a
          href="https://cds.climate.copernicus.eu/cdsapp#!/dataset/insitu-gridded-observations-global-and-regional?tab=overview"
          target="_blank"
          rel="noopener"
        >
          Copernicus
        </a>
        de 2001 à {{ result.lastKnownYear }}.
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import Plotly from "plotly.js-dist-min";

const emit = defineEmits([
  "updateResult",
]);

const props = defineProps<{
  loading: boolean,
  result: {
    waterPrice: { price: number, year: number, division: "national" | "departemental" | "communal" }, // price : €/m3
    lastKnownYear: string,
    waterRecoverableQuantity: Record<string, number>, // L
    savingForLastKnownYear?: number, // €/m3/year
    idealCapacity?: number, // L/year
    waterNeeds?: number, // L/year
    roofSurfaceArea: number, // m²
    gardenSurfaceArea: number, // m²
    vegetableSurfaceArea: number, // m²
    evolutionStockWater: number[], // L
    consumptionByTapWater: number[], // L
    driestYear: string,
    wettestYear: string
  } | null,
}>();

const graphToDisplay: Ref<"recently" | "driest" | "wettest"> = ref("recently");

const getDivisionForWaterPrice = computed(() => {
  const division = props.result?.waterPrice.division;
  if (division === "communal") return "dans votre commune";
  if (division === "departemental") return "dans votre département";
  return "au niveau national";
});

const selectedScenario = (scenario: "recently" | "driest" | "wettest") => {
  graphToDisplay.value = scenario;
  emit("updateResult", scenario);
};

const drawGraph = () => {
  if (props.result?.waterNeeds && props.result.waterRecoverableQuantity) {
    let waterRecovery = {
      x: ["Janv", "Févr", "Mars", "Avril", "Mai", "Juin", "Juil", "Août", "Sept", "Oct", "Nov", "Déc"],
      y: Object.keys(props.result.waterRecoverableQuantity).map(key => props.result.waterRecoverableQuantity[key]),
      hovertemplate: "%{y} L<extra></extra>",
      type: "bar",
      name: `Précipitations enregistrées`,

    };
    if (graphToDisplay.value === "recently") {
      waterRecovery = {
        ...waterRecovery,
        marker: {
          color: "#29235c",
        },
      };
    } else if (graphToDisplay.value === "driest") {
      waterRecovery = {
        ...waterRecovery,
        marker: {
          color: "#af6708",
          opacity: 0.6,
        },
      };
    } else {
      waterRecovery = {
        ...waterRecovery,
        marker: {
          color: "#085421",
          opacity: 0.6,
        },
      };
    }

    const waterNeeded = {
      x: ["Janv", "Févr", "Mars", "Avril", "Mai", "Juin", "Juil", "Août", "Sept", "Oct", "Nov", "Déc"],
      y: [
        0,
        0,
        0,
        Math.round(props.result.waterNeeds / 7),
        Math.round(props.result.waterNeeds / 7),
        Math.round(props.result.waterNeeds / 7),
        Math.round(props.result.waterNeeds / 7),
        Math.round(props.result.waterNeeds / 7),
        Math.round(props.result.waterNeeds / 7),
        Math.round(props.result.waterNeeds / 7),
        0,
        0,
      ],
      hovertemplate: "%{y} L<extra></extra>",
      type: "bar",
      name: "Besoin en eau par mois",
      marker: {
        color: "#009fe3",
      },
    };

    const evolutionStockWater = {
      x: ["Janv", "Févr", "Mars", "Avril", "Mai", "Juin", "Juil", "Août", "Sept", "Oct", "Nov", "Déc"],
      y: props.result?.evolutionStockWater,
      hovertemplate: "%{y} L<extra></extra>",
      type: "scatter",
      name: "Évolution du stockage au sein du récupérateur",
      marker: {
        color: "#9b093e",
        opacity: 0.6,
      },
      line: {
        dash: "dot",
        shape: "spline",
      },
    };
    const evolutionUseTapWater = {
      x: ["Janv", "Févr", "Mars", "Avril", "Mai", "Juin", "Juil", "Août", "Sept", "Oct", "Nov", "Déc"],
      y: props.result?.consumptionByTapWater,
      hovertemplate: "%{y} L<extra></extra>",
      type: "scatter",
      name: "Usage de l'eau courante pour palier à l'insuffisance du récupérateur",
      marker: {
        color: "#9b8309",
        opacity: 0.6,
      },
      line: {
        dash: "dot",
        shape: "spline",
      },
    };

    const layout = {
      font: {
        size: 10,
      },
      xaxis: {
        tickangle: -40,
        tickfont: {
          size: 10,
        },
      },
      yaxis: { range: [0, 6000] },
      barmode: "group",
      bargap: 0.15,
      bargroupgap: 0.1,
      showlegend: true,
      legend: { orientation: "h" },
      height: 400,
      margin: {
        l: 50,
        r: 50,
        b: 0,
        t: 40,
        pad: 2,
      },
    };
    return {
      data: [
        waterRecovery,
        waterNeeded,
        evolutionStockWater,
        evolutionUseTapWater,
      ],
      layout,
    };
  }
  return null;
};

watch(() => props.result, () => {
  // Needed to always have the last result computed
  const res = drawGraph();
  if (res) {
    const refChart = document.getElementById("refChart");
    Plotly.react(
      refChart!,
      res.data,
      res.layout,
      {
        showTips: false,
        modeBarButtonsToRemove: [
          "zoom2d",
          "zoomIn2d",
          "zoomOut2d",
          "pan2d",
          "select2d",
          "lasso2d",
          "toImage",
          "autoScale2d",
        ],
        displaylogo: false,
      },
    );
  }
});
</script>
