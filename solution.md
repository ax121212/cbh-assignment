Part 1 - Ticket Breakdown
As inferred from the description above I understand that each Agent has id and name mapping in the database and the requirement is to get a custom report using `generateReport` utility which currently accepts response from `getShiftsByFacility` that takes in a facility ID and returns back shifts for the entire facility of all agents. The custom report is expected to provide stats only for specific Agent IDs and not the entire facility by using custom ID's they wish to assign to each agent.

Implementation detail -

Update the schema of Agent table and add support for a custom ID against each row.
Create a utility to map each agent with a new custom ID in the format as chosen by the facility.
Update `getShiftsByFacility` utility to accept facility ID and also a list of custom Agent ID's so as to return a filtered list/array of shifts (assuming the list/arrray consists of objects having `agentId` and `shiftDetails`)
Then passing the filtered array/list to `getShiftsByFacility`

Tickets created - 

1) Agent Table Database Schema Update
2) Utility for updating Custom Agent against each Agent
3) Refactoring getShiftsByFacility to accept array of Agent ID's to return back filtered shift list

Part 2 - Refactoring code -
1) Unit test -
const { deterministicPartitionKey } = require("./dpk");


describe("deterministicPartitionKey", () => {
  it("Returns the literal '0' when given no input", () => {
    const trivialKey = deterministicPartitionKey();
    expect(trivialKey).toBe("0");
  });
  it("trivial key is generated by crypto, when event is passed but partitionKey is not present", () => {
    const trivialKey = deterministicPartitionKey({});
    expect(trivialKey.length).not.toBe(0);
  });
  it("trivial key is partitionKey, when event is passed and partitionKey is present", () => {
    const trivialKey = deterministicPartitionKey({partitionKey : "mockPartitionKey"});
    expect(trivialKey).toBe("mockPartitionKey");
  });
  it("trivial key is partitionKey, when event is passed and partitionKey is present but not string", () => {
    const trivialKey = deterministicPartitionKey({partitionKey : {}});
    expect(typeof trivialKey).toBe("string");
  });
  it("trivial key is generated by crypto, when event is passed and partitionKey is present, but length is more than 256", () => {
    let mockPartitionKey = "a";
    for(let i=0;i<256;i++){
      mockPartitionKey+="a";
    }
    const trivialKey = deterministicPartitionKey({partitionKey : mockPartitionKey});
    expect(trivialKey).not.toBe(mockPartitionKey);
  });
});


2) Refactored Code -

const crypto = require("crypto");


exports.deterministicPartitionKey = (event) => {
  const TRIVIAL_PARTITION_KEY = "0", MAX_PARTITION_KEY_LENGTH = 256;
  let candidate;


  if (event) {
    candidate = event.partitionKey ? event.partitionKey : crypto.createHash("sha3-512").update(JSON.stringify(event)).digest("hex");
    if (typeof candidate !== "string") {
      candidate = JSON.stringify(candidate);
    }
    if (candidate.length > MAX_PARTITION_KEY_LENGTH) {
      candidate = crypto.createHash("sha3-512").update(candidate).digest("hex");
    }  
  } else {
    candidate = TRIVIAL_PARTITION_KEY;
  }


  return candidate;
};

3) Refactoring explained -
In the first if block we were checking the value of event and setting the value of candidate accordingly. In the next if condition we were checking the value of candidate and running operations on the same accordingly which seemed redundant. Hence we checked the value of event in the beginning and combined all conditions in a big if else condition, inside which we have smaller conditions which include transformation in values on the basis of initial value being set using the ternary operator or else if event doesnt exist we set the value as TRIVIAL_PARTITION_KEY

