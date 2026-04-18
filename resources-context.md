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
    "requires": ["auth/2fa"],
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

{
  "send_2fa": {
    "route": "/2fa/send",
    "method": "POST",
    "required_args": ["to"],
    "description": "",
    "produces": ["unique 2fa id"]
  },
  "verify_2fa": {
    "route": "/2fa/verify",
    "method": "POST",
    "required_args": ["userid","unique_2fa_id"],
    "description": "",
    "constraints": ["requires unique 2fa id and in question intent consistencies/relatable and prevent 2fa leak withing concurrent 2fa sessions"]
  }
}
