{
  "create_transfer": {
    "route": "/transfer/create",
    "method": "POST",
    "required_args": ["from", "to", "amount"],
    "description": "Create a transfer that requires approval",
    "produces": ["transferId"]
  },
  "approve_transfer": {
    "route": "/transfer/approve",
    "method": "POST",
    "required_args": ["transferId"],
    "description": "Approve a pending transfer",
    "requires": ["auth"],
    "constraints": ["must_be_owner_or_admin", "must_be_pending"]
  },
  "dispute_transfer": {
    "route": "/transfer/dispute",
    "method": "POST",
    "required_args": ["transferId", "reason"],
    "constraints": ["must_be_participant_or_admin"]
  },
  "file_for_an_approve_proceeding": {
    "route": "/transfer/subscribe",
    "method": "POST",
    "required_args": ["transferId", "note"],
    "description": "Submit proof of conveyance",
    "constraints": ["must_be_buyer"]
  }
}
