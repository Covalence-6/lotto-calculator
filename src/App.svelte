<script lang="ts">
  import { onMount } from 'svelte'

  const numberHeaders: string[] = ['"NUMBER DRAWN 1"', '"NUMBER DRAWN 2"', '"NUMBER DRAWN 3"', '"NUMBER DRAWN 4"', '"NUMBER DRAWN 5"', '"NUMBER DRAWN 6"']

  let previousDraws: {[key: string]: string}[] = $state([])
  let selectedNumbers: number[] = $state([0, 0, 0, 0, 0, 0])

  let selectedNumbersLastDrawn: number[] = $derived(
    [numberLastDrawn(selectedNumbers[0]),
    numberLastDrawn(selectedNumbers[1]),
    numberLastDrawn(selectedNumbers[2]),
    numberLastDrawn(selectedNumbers[3]),
    numberLastDrawn(selectedNumbers[4]),
    numberLastDrawn(selectedNumbers[5])
    ]
  )

  let selectedNumbersPercentDrawn: number[] = $derived(
    [numberPercentDrawn(selectedNumbers[0]),
    numberPercentDrawn(selectedNumbers[1]),
    numberPercentDrawn(selectedNumbers[2]),
    numberPercentDrawn(selectedNumbers[3]),
    numberPercentDrawn(selectedNumbers[4]),
    numberPercentDrawn(selectedNumbers[5])
    ]
  )

  let selectionSum: number = $derived(selectedNumbers.reduce((partialSum, a) => partialSum + a, 0))

  let oddNumbers: number = $derived.by(() => {
    let oddTotal = 0

    selectedNumbers.forEach((num) => {
      if (num%2 !== 0) {
        oddTotal++
      }
    })

    return oddTotal
  })

  let lowNumbers: number = $derived.by(() => {
    let lowTotal = 0

    selectedNumbers.forEach((num) => {
      if (num < 25) {
        lowTotal++
      }
    })

    return lowTotal
  })

  function csvToArr(stringVal: string, splitter: string) {
    const [keys, ...rest] = stringVal
      .trim()
      .split("\n")
      .map((item) => item.replace("\r", "").split(splitter)) // Depending on data source csv rows may have carriage return characters

    const formedArr = rest.map((item) => {
      const obj: {[key: string]: string} = {}
      keys.forEach((key: string, index: number) => (obj[key] = item.at(index) ?? ''))
      return obj
    })

    return formedArr
  }

  function numberLastDrawn(targetNumber: number): number {
    let targetNumberFound: boolean = false
    let lastDrawn: number = 0

    for (let i = 0; i < previousDraws.length; i++) {
      const draw = previousDraws[i]

      for (const [key, value] of Object.entries(draw)) {
        if (numberHeaders.includes(key) && Number(value) === targetNumber) {
          targetNumberFound = true
          break
        }
      }

      if (targetNumberFound) {
        lastDrawn = previousDraws.length - Number(draw['"DRAW NUMBER"']) + 1
        break
      }
    }
      
    return lastDrawn
  }

  function numberPercentDrawn(targetNumber: number): number {
    let timesDrawn: number = 0

    for (let i = 0; i < previousDraws.length; i++) {
      const draw = previousDraws[i]

      for (const [key, value] of Object.entries(draw)) {
        if (numberHeaders.includes(key) && Number(value) === targetNumber) {
          timesDrawn++
          break
        }
      }
    }
      
    const percentageDrawn = timesDrawn / previousDraws.length * 100

    if (timesDrawn === 0) {
      return timesDrawn
    }

    return Number(percentageDrawn.toFixed(4))
  }

  onMount(() => {
    const form = document.querySelector("#csvForm") as HTMLFormElement
    const csvFileInput = document.querySelector("#csvInput") as HTMLInputElement

    form?.addEventListener("submit", function (event: Event) {
      event.preventDefault()

      const files = csvFileInput?.files
      const file = (files as FileList)[0]
      const reader = new FileReader()

      reader.onload = function (e: Event) {
        const csvArray = csvToArr(reader.result as string, ",")
        previousDraws = csvArray.reverse()
      }

      reader.readAsText(file)
    })
  })

</script>

<main>
  <form id="csvForm">
    <input type="file" id="csvInput" accept=".csv" />
    <input type="submit" value="Submit" />
  </form>

  <table>
    <thead>
      <tr>
        <th>Numbers</th>
        <th>Selected Numbers</th>
        <th>Percent Drawn (%)</th>
        <th>Last Drawn (Weeks)</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>First Number</th>
        <td>
          <input type="number" id="firstSelectedNumber" min="1" max="49" step="1" bind:value={selectedNumbers[0]}/>
        </td>
        <td>{selectedNumbersPercentDrawn[0]}</td>
        <td>{selectedNumbersLastDrawn[0]}</td>
      </tr>
      <tr>
        <th>Second Number</th>
        <td>
          <input type="number" id="secondSelectedNumber" min="1" max="49" step="1" bind:value={selectedNumbers[1]}/>
        </td>
        <td>{selectedNumbersPercentDrawn[1]}</td>
        <td>{selectedNumbersLastDrawn[1]}</td>
      </tr>
      <tr>
        <th>Third Number</th>
        <td>
          <input type="number" id="thirdSelectedNumber" min="1" max="49" step="1" bind:value={selectedNumbers[2]}/>
        </td>
        <td>{selectedNumbersPercentDrawn[2]}</td>
        <td>{selectedNumbersLastDrawn[2]}</td>
      </tr>
      <tr>
        <th>Fourth Number</th>
        <td>
          <input type="number" id="fourthSelectedNumber" min="1" max="49" step="1" bind:value={selectedNumbers[3]}/>
        </td>
        <td>{selectedNumbersPercentDrawn[3]}</td>
        <td>{selectedNumbersLastDrawn[3]}</td>
      </tr>
      <tr>
        <th>Fifth Number</th>
        <td>
          <input type="number" id="fifthSelectedNumber" min="1" max="49" step="1" bind:value={selectedNumbers[4]}/>
        </td>
        <td>{selectedNumbersPercentDrawn[4]}</td>
        <td>{selectedNumbersLastDrawn[4]}</td>
      </tr>
      <tr>
        <th>Sixth Number</th>
        <td>
          <input type="number" id="sixthSelectedNumber" min="1" max="49" step="1" bind:value={selectedNumbers[5]}/>
        </td>
        <td>{selectedNumbersPercentDrawn[5]}</td>
        <td>{selectedNumbersLastDrawn[5]}</td>
      </tr>
    </tbody>
  </table>

  <table>
    <thead>
      <tr>
        <th>Statistic</th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <th>Total Selection Sum</th>
        <td>{selectionSum}</td>
      </tr>
      <tr>
        <th>Odd Numbers</th>
        <td>{oddNumbers}</td>
      </tr>
      <tr>
        <th>Low Numbers</th>
        <td>{lowNumbers}</td>
      </tr>
    </tbody>
  </table>
</main>

<style></style>
