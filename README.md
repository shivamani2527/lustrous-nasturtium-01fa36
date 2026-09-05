<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AyushCare - Smart OPD & Clinical Case-Taking Network</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
  <!-- QR Code Generator CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
  <!-- Html5-QRCode Camera Scanner CDN -->
  <script src="https://unpkg.com/html5-qrcode@2.3.8/html5-qrcode.min.js"></script>
  
  <!-- Firebase App & Realtime Database SDK CDNs -->
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>

  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800&display=swap');
    body { font-family: 'Inter', sans-serif; }
    .tab-active-patient { border-bottom: 3px solid #0d9488; color: #0f766e; font-weight: 700; }
    .tab-active-hospital { border-bottom: 3px solid #d97706; color: #b45309; font-weight: 700; }

    @media print {
      body * { visibility: hidden; }
      #printReportSection, #printReportSection * { visibility: visible; }
      #printReportSection {
        position: absolute; left: 0; top: 0; width: 100%; margin: 0; padding: 24px;
        background: white; border: none !important; box-shadow: none !important;
      }
      .no-print { display: none !important; }
    }
  </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col antialiased">

  <!-- ================= AUTHENTICATION / LOGIN & SIGNUP SCREEN ================= -->
  <div id="authScreen" class="fixed inset-0 z-[100] bg-slate-900/90 backdrop-blur-md flex items-center justify-center p-4">
    <div class="bg-white w-full max-w-md rounded-3xl p-7 md:p-8 shadow-2xl space-y-5 relative overflow-hidden border border-slate-100">
      <div class="absolute -right-12 -top-12 w-36 h-36 bg-teal-500/10 rounded-full blur-2xl"></div>
      
      <!-- Brand & Header -->
      <div class="text-center space-y-1">
        <div class="inline-flex p-3 rounded-2xl bg-teal-50 text-teal-700 text-2xl shadow-sm mb-1">
          <i class="fa-solid fa-hospital-user"></i>
        </div>
        <h2 class="text-2xl font-black text-slate-800 tracking-tight">AyushCare Portal</h2>
        <p class="text-xs text-slate-500">Smart OPD & Electronic Clinical Intake System</p>
      </div>

      <!-- Auth Action: Login vs Signup -->
      <div class="flex border-b border-slate-200">
        <button type="button" onclick="setAuthTab('login')" id="tabBtnLogin" class="flex-1 pb-2.5 text-xs font-bold text-teal-700 border-b-2 border-teal-600 transition">
          Sign In
        </button>
        <button type="button" onclick="setAuthTab('signup')" id="tabBtnSignup" class="flex-1 pb-2.5 text-xs font-semibold text-slate-400 hover:text-slate-700 transition">
          Create New Account
        </button>
      </div>

      <!-- Role Selector -->
      <div class="grid grid-cols-2 p-1 bg-slate-100 rounded-xl text-xs font-semibold text-slate-600">
        <button type="button" onclick="setAuthRole('patient')" id="authRolePatient" class="py-2.5 rounded-lg bg-white shadow-sm text-teal-800 font-bold transition">
          <i class="fa-solid fa-user-circle mr-1.5"></i> Patient
        </button>
        <button type="button" onclick="setAuthRole('hospital')" id="authRoleHospital" class="py-2.5 rounded-lg text-slate-500 hover:text-slate-800 transition">
          <i class="fa-solid fa-hospital mr-1.5"></i> Hospital Staff
        </button>
      </div>

      <!-- Form Error / Feedback Banner -->
      <div id="authAlertBox" class="hidden p-3 rounded-xl text-xs flex items-center space-x-2">
        <i class="fa-solid fa-circle-exclamation text-base"></i>
        <span id="authAlertText" class="font-medium"></span>
      </div>

      <!-- Auth Form -->
      <form id="authForm" onsubmit="handleAuthSubmit(event)" class="space-y-3.5">
        
        <!-- Hospital Info Group -->
        <div id="hospitalAuthGroup" class="hidden space-y-2">
          <div class="space-y-1">
            <label class="block text-[11px] font-bold text-slate-700">Hospital / Medical Center Name *</label>
            <input type="text" id="authHospitalName" placeholder="e.g., St. James Apex Hospital" value="St. James Hospital" class="w-full px-3.5 py-2.5 border rounded-xl text-xs bg-slate-50 focus:bg-white outline-none" />
          </div>
          <div class="space-y-1" id="hospLogoBox">
            <label class="block text-[11px] font-bold text-slate-700">Hospital Logo / Photo</label>
            <input type="file" id="authHospitalLogoInput" accept="image/*" onchange="previewHospitalLogo(event)" class="w-full text-xs text-slate-500 border p-1 rounded-xl" />
          </div>
        </div>

        <!-- Full Name (Only for Signup) -->
        <div id="signupNameGroup" class="hidden space-y-1">
          <label class="block text-[11px] font-bold text-slate-700" id="nameFieldLabel">Full Name *</label>
          <input type="text" id="authFullName" placeholder="e.g., Rahul Sharma" class="w-full px-3.5 py-2.5 border rounded-xl text-xs bg-slate-50 focus:bg-white outline-none" />
        </div>

        <div class="space-y-1">
          <label class="block text-[11px] font-bold text-slate-700" id="authIdentifierLabel">Patient Email / ABHA ID *</label>
          <input type="text" id="authIdentifier" required placeholder="Enter your email / ID" value="rahul.sharma@medmail.com" class="w-full px-3.5 py-2.5 border rounded-xl text-xs bg-slate-50 focus:bg-white outline-none" />
        </div>

        <div class="space-y-1">
          <label class="block text-[11px] font-bold text-slate-700">Password *</label>
          <input type="password" id="authPassword" required placeholder="Enter your password" value="123456" class="w-full px-3.5 py-2.5 border rounded-xl text-xs bg-slate-50 focus:bg-white outline-none" />
        </div>

        <button type="submit" id="authSubmitBtn" class="w-full py-3 bg-teal-700 hover:bg-teal-800 text-white font-bold rounded-xl text-xs shadow-lg transition flex items-center justify-center space-x-2">
          <span>Sign In</span> <i class="fa-solid fa-arrow-right text-[10px]"></i>
        </button>
      </form>
    </div>
  </div>

  <!-- ================= TOP APPLICATION HEADER ================= -->
  <header class="bg-teal-800 text-white sticky top-0 z-40 shadow-md">
    <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center gap-3">
      <div class="flex items-center space-x-3">
        <div class="bg-white text-teal-800 p-2 rounded-xl font-bold text-lg shadow">
          <i class="fa-solid fa-notes-medical"></i>
        </div>
        <div>
          <div class="flex items-center space-x-2">
            <h1 class="font-bold text-base md:text-lg leading-tight" id="headerMainTitle">AyushCare OPD System</h1>
            <span id="headerRoleBadge" class="bg-teal-950 text-teal-200 text-[10px] font-extrabold px-2.5 py-0.5 rounded-full uppercase tracking-wider"></span>
          </div>
          <p class="text-[11px] text-teal-200" id="headerSubtitle">Patient Case-Taking & Diagnostic Vault</p>
        </div>
      </div>

      <!-- Action Items -->
      <div class="flex items-center gap-3">
        <div class="text-right hidden sm:block">
          <span class="text-xs font-bold block leading-tight" id="userGreetingName">User</span>
          <span class="text-[10px] text-teal-300" id="userGreetingRole">Logged in</span>
        </div>

        <button onclick="downloadSelf()" class="bg-amber-500 hover:bg-amber-600 text-slate-900 px-3.5 py-1.5 rounded-xl text-xs font-bold transition flex items-center shadow">
          <i class="fa-solid fa-download mr-1.5"></i> Save App
        </button>

        <button onclick="logoutSession()" title="Logout" class="bg-teal-950 hover:bg-rose-700 text-white px-3 py-1.5 rounded-xl text-xs font-bold transition flex items-center space-x-1.5">
          <i class="fa-solid fa-power-off"></i> <span class="hidden sm:inline">Logout</span>
        </button>
      </div>
    </div>
  </header>

  <!-- ================= MAIN WORKSPACE ================= -->
  <main class="max-w-7xl mx-auto p-4 md:p-6 flex-1 w-full space-y-6">

    <!-- ============================================================= -->
    <!--                      1. PATIENT MODE                         -->
    <!-- ============================================================= -->
    <div id="patientSection" class="space-y-6 hidden">
      
      <!-- Patient Navigation Tabs -->
      <div class="bg-white rounded-2xl shadow-sm border border-slate-200 flex overflow-x-auto p-1">
        <button onclick="switchPatientTab(1)" id="ptTab1" class="tab-active-patient px-5 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-file-invoice mr-2"></i> 1. Medical Registration & Pass
        </button>
        <button onclick="switchPatientTab(2)" id="ptTab2" class="px-5 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-user-doctor mr-2"></i> 2. Book Specialist Doctor
        </button>
        <button onclick="switchPatientTab(3)" id="ptTab3" class="px-5 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-folder-medical mr-2"></i> 3. My Medical Reports (<span id="reportCountBadge">0</span>)
        </button>
        <button onclick="switchPatientTab(4)" id="ptTab4" class="px-5 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-calendar-check mr-2"></i> 4. My Bookings (<span id="patientBookingCountBadge">0</span>)
        </button>
      </div>

      <!-- Tab 1: Comprehensive Medical Form with Full Name and Photo -->
      <div id="ptPage1" class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        <!-- Medical Form Column (2 Cols) -->
        <div class="lg:col-span-2 bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
          
          <div class="bg-[#1e4d79] text-white p-4 flex justify-between items-center">
            <h2 class="text-lg font-black tracking-wide uppercase">PATIENT MEDICAL FORM</h2>
            <div class="flex items-center space-x-3 bg-slate-900/40 px-3 py-1.5 rounded-xl border border-white/10">
              <img id="patientHeaderAvatar" src="https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=100&auto=format&fit=crop&q=80" alt="Patient Photo" class="w-10 h-10 rounded-full object-cover border-2 border-teal-400" />
              <div class="text-right text-xs">
                <span class="font-bold block tracking-wider text-white" id="patientProfileDisplay">Rahul Sharma</span>
                <span class="text-teal-200 text-[10px]">Verified Patient</span>
              </div>
            </div>
          </div>

          <form id="patientForm" onsubmit="generatePatientQR(event)" class="p-6 space-y-5">
            
            <!-- SECTION 1: PATIENT INFORMATION -->
            <div class="space-y-3">
              <div class="bg-amber-500 text-slate-900 font-black text-xs px-3 py-1.5 tracking-wider uppercase rounded-sm">
                PATIENT INFORMATION
              </div>
              <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Patient's Full Name *</label>
                  <input type="text" id="pFullName" required value="Rahul Sharma" oninput="updatePatientDisplayName()" class="w-full p-2 border rounded-lg text-xs bg-slate-50 focus:bg-white focus:ring-1 focus:ring-teal-500 outline-none" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Age *</label>
                  <input type="number" id="pAge" required value="32" class="w-full p-2 border rounded-lg text-xs bg-slate-50 focus:bg-white outline-none" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Blood Group *</label>
                  <select id="pBloodGroup" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none font-bold text-slate-800">
                    <option selected>O+</option><option>O-</option><option>A+</option><option>A-</option><option>B+</option><option>B-</option><option>AB+</option><option>AB-</option>
                  </select>
                </div>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Patient Profile Photo (Upload)</label>
                  <input type="file" id="patientPhotoInput" accept="image/*" onchange="previewPatientPhoto(event)" class="w-full text-xs text-slate-500 file:mr-2 file:py-1 file:px-2 file:rounded file:border-0 file:text-xs file:bg-teal-50 file:text-teal-700 border p-1 rounded-lg" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Phone Number *</label>
                  <input type="tel" id="pPhone" required value="+91 98765 43210" class="w-full p-2 border rounded-lg text-xs bg-slate-50 focus:bg-white outline-none" />
                </div>
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Date of Birth *</label>
                  <input type="date" id="pDOB" required value="1994-06-15" class="w-full p-2 border rounded-lg text-xs bg-slate-50 focus:bg-white outline-none" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Email Address *</label>
                  <input type="email" id="pEmail" required value="rahul.sharma@medmail.com" class="w-full p-2 border rounded-lg text-xs bg-slate-50 focus:bg-white outline-none" />
                </div>
              </div>
            </div>

            <!-- SECTION 2: EMERGENCY CONTACT INFORMATION -->
            <div class="space-y-3">
              <div class="bg-amber-500 text-slate-900 font-black text-xs px-3 py-1.5 tracking-wider uppercase rounded-sm">
                EMERGENCY CONTACT INFORMATION
              </div>
              <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Name *</label>
                  <input type="text" id="pEmergName" required value="Sunita Sharma" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Relationship *</label>
                  <input type="text" id="pEmergRel" required value="Spouse" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Phone Number *</label>
                  <input type="tel" id="pEmergPhone" required value="+91 91234 56789" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                </div>
              </div>
            </div>

            <!-- SECTION 3: INSURANCE INFORMATION -->
            <div class="space-y-3">
              <div class="bg-amber-500 text-slate-900 font-black text-xs px-3 py-1.5 tracking-wider uppercase rounded-sm">
                INSURANCE INFORMATION
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Insurance Provider</label>
                  <input type="text" id="pInsuranceProv" value="Star Health / National Ayush Scheme" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Policy Number / ABHA ID</label>
                  <input type="text" id="pPolicyNum" value="ABHA-9821-4402-1189" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                </div>
              </div>
            </div>

            <!-- SECTION 4: MEDICAL HISTORY -->
            <div class="space-y-3">
              <div class="bg-amber-500 text-slate-900 font-black text-xs px-3 py-1.5 tracking-wider uppercase rounded-sm">
                MEDICAL HISTORY
              </div>
              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-2">Pre-existing Conditions (Check all that apply)</label>
                  <div class="space-y-1.5 text-xs">
                    <label class="flex items-center space-x-2 cursor-pointer">
                      <input type="checkbox" id="condDiabetes" class="rounded text-teal-600" />
                      <span>Diabetes</span>
                    </label>
                    <label class="flex items-center space-x-2 cursor-pointer">
                      <input type="checkbox" id="condHypertension" checked class="rounded text-teal-600" />
                      <span>Hypertension</span>
                    </label>
                    <label class="flex items-center space-x-2 cursor-pointer">
                      <input type="checkbox" id="condAsthma" class="rounded text-teal-600" />
                      <span>Asthma</span>
                    </label>
                    <label class="flex items-center space-x-2 cursor-pointer">
                      <input type="checkbox" id="condCancer" class="rounded text-teal-600" />
                      <span>Cancer</span>
                    </label>
                    <div class="flex items-center space-x-2 pt-1">
                      <label class="flex items-center space-x-2 cursor-pointer whitespace-nowrap">
                        <input type="checkbox" id="condOtherCheck" class="rounded text-teal-600" />
                        <span>Other:</span>
                      </label>
                      <input type="text" id="condOtherText" placeholder="Specify condition" class="w-full p-1 border rounded text-xs" />
                    </div>
                  </div>
                </div>

                <div class="space-y-3">
                  <div>
                    <label class="block text-[11px] font-bold text-slate-700 mb-1">Known Allergies:</label>
                    <input type="text" id="pAllergiesList" value="Penicillin, Sulfa drugs" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                  </div>
                  <div>
                    <label class="block text-[11px] font-bold text-slate-700 mb-1">Active Medications:</label>
                    <input type="text" id="pMedicationsList" value="Amlodipine 5mg (OD)" class="w-full p-2 border rounded-lg text-xs bg-slate-50 outline-none" />
                  </div>
                </div>
              </div>
            </div>

            <!-- SECTION 5: PURPOSE OF VISIT / DISEASE -->
            <div class="space-y-2">
              <div class="bg-amber-500 text-slate-900 font-black text-xs px-3 py-1.5 tracking-wider uppercase rounded-sm">
                DIAGNOSIS / DISEASE / REASON FOR VISIT
              </div>
              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Describe symptoms, suspected disease or reason for visit *</label>
                <textarea id="pVisitReason" required rows="3" class="w-full p-2.5 border rounded-lg text-xs bg-slate-50 outline-none focus:bg-white" placeholder="e.g. Chronic Sciatica / Lumbar Spondylosis with lower back radiation...">Chronic Lumbar Spondylosis with Lower Back Nerve Compression</textarea>
              </div>
            </div>

            <button type="submit" class="w-full bg-teal-700 hover:bg-teal-800 text-white font-bold py-3 rounded-xl text-xs shadow-md transition flex items-center justify-center space-x-2">
              <i class="fa-solid fa-qrcode"></i> <span>Update Record & Generate Live QR Pass</span>
            </button>
          </form>
        </div>

        <!-- Live QR Pass Card Column (1 Col) -->
        <div class="space-y-4">
          <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm flex flex-col items-center justify-center text-center space-y-4">
            <div>
              <span class="bg-teal-100 text-teal-800 text-[10px] font-extrabold uppercase px-2.5 py-1 rounded-full">Touchless Fast-Track OPD</span>
              <h3 class="font-black text-slate-800 text-base mt-2">Live Dynamic QR Pass</h3>
              <p class="text-xs text-slate-500">Present this QR at any participating hospital for instant intake.</p>
            </div>
            
            <div id="qrcodeBox" class="p-4 bg-white border-2 border-teal-500 rounded-2xl shadow-sm min-h-[190px] flex items-center justify-center"></div>

            <div id="qrMeta" class="text-xs text-slate-600 w-full bg-slate-50 p-3 rounded-xl border border-slate-100 space-y-1">
              <p class="font-bold text-slate-800" id="qrPatientTag">Rahul Sharma</p>
              <p class="text-[10px] text-slate-400 font-mono" id="qrTimestampTag"></p>
            </div>
          </div>
        </div>
      </div>

      <!-- Tab 2: Doctor Directory & Appointments across All Specialties & Hospitals -->
      <div id="ptPage2" class="space-y-4 hidden">
        <div class="flex flex-col md:flex-row justify-between md:items-center gap-3 bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
          <div>
            <h2 class="font-bold text-slate-800 text-base">Book a Specialist (Cross-Hospital Directory)</h2>
            <p class="text-xs text-slate-500">Browse doctors from different hospitals by specialty and check live availability.</p>
          </div>
          <div class="flex flex-wrap gap-2">
            <!-- Filter by Department -->
            <select id="filterDepartment" onchange="renderPatientDoctors()" class="border p-2 rounded-xl text-xs bg-slate-50 outline-none font-medium">
              <option value="All">All Specialties / Departments</option>
              <option value="General Practice">General Practice</option>
              <option value="Gastroenterology">Gastroenterology</option>
              <option value="Cardiology">Cardiology</option>
              <option value="Pediatrics">Pediatrics</option>
              <option value="Endocrinology">Endocrinology</option>
              <option value="Orthopedic">Orthopedic</option>
              <option value="Pathology">Pathology</option>
              <option value="Radiology">Radiology</option>
              <option value="Psychiatry">Psychiatry</option>
              <option value="Ophthalmology">Ophthalmology</option>
              <option value="Neurology">Neurology</option>
              <option value="Pulmonology">Pulmonology</option>
              <option value="Otolaryngology (ENT)">Otolaryngology (ENT)</option>
              <option value="Urology">Urology</option>
              <option value="Kayachikitsa (Internal Med)">Kayachikitsa (Ayurveda)</option>
              <option value="Panchakarma">Panchakarma</option>
            </select>

            <!-- Filter by Hospital -->
            <select id="filterHospital" onchange="renderPatientDoctors()" class="border p-2 rounded-xl text-xs bg-slate-50 outline-none font-medium">
              <option value="All">All Hospitals</option>
            </select>
          </div>
        </div>

        <!-- Doctors Grid -->
        <div id="patientDoctorsList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"></div>
      </div>

      <!-- Tab 3: Medical Reports Vault -->
      <div id="ptPage3" class="space-y-4 hidden">
        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
          <div class="flex justify-between items-center">
            <div>
              <h2 class="font-bold text-slate-800 text-base">Patient Medical Vault (Diagnostic & Scan Reports)</h2>
              <p class="text-xs text-slate-500">Official lab & radiology results uploaded directly by hospital departments.</p>
            </div>
          </div>

          <div id="patientReportsContainer" class="grid grid-cols-1 md:grid-cols-2 gap-4"></div>
        </div>
      </div>

      <!-- Tab 4: Patient View Booked Appointments -->
      <div id="ptPage4" class="space-y-4 hidden">
        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
          <div class="flex justify-between items-center">
            <div>
              <h2 class="font-bold text-slate-800 text-base">My Booked Specialist Appointments</h2>
              <p class="text-xs text-slate-500">View all your upcoming and past doctor appointments across hospitals.</p>
            </div>
          </div>
          <div class="overflow-x-auto rounded-xl border border-slate-200">
            <table class="w-full text-left text-xs text-slate-600">
              <thead class="bg-slate-100 text-slate-800 uppercase text-[11px] font-bold">
                <tr>
                  <th class="p-3">Hospital & Department</th>
                  <th class="p-3">Specialist Doctor</th>
                  <th class="p-3">Slot / Date</th>
                  <th class="p-3 text-center">Status</th>
                </tr>
              </thead>
              <tbody id="patientAppointmentsBody" class="divide-y divide-slate-100"></tbody>
            </table>
          </div>
        </div>
      </div>
    </div>


    <!-- ============================================================= -->
    <!--                     2. HOSPITAL MODE                         -->
    <!-- ============================================================= -->
    <div id="hospitalSection" class="space-y-6 hidden">
      
      <!-- Hospital Mode Top Banner -->
      <div class="bg-amber-500 text-slate-950 p-4 rounded-2xl flex flex-wrap items-center justify-between gap-3 shadow-sm">
        <div class="flex items-center space-x-3">
          <img id="hospitalBannerLogo" src="https://images.unsplash.com/photo-1519494026892-80bbd2d6fd0d?w=120&auto=format&fit=crop&q=80" alt="Hospital Logo" class="w-12 h-12 rounded-xl object-cover border-2 border-slate-950 bg-white" />
          <div>
            <h3 class="text-sm font-black uppercase tracking-wider" id="hospitalModeBannerTitle">ST. JAMES HOSPITAL OPD DESK</h3>
            <p class="text-xs text-slate-900 font-medium">Smart Clinical Triage, Live QR Ingestion & Roster Management</p>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <span class="bg-slate-900 text-amber-400 text-xs px-3 py-1 rounded-full font-bold">OPD Desk Active</span>
        </div>
      </div>

      <!-- Hospital Navigation Tabs -->
      <div class="bg-white rounded-2xl shadow-sm border border-slate-200 flex overflow-x-auto p-1">
        <button onclick="switchHospitalTab('ingestion')" id="hospTabIngestion" class="tab-active-hospital px-4 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-qrcode mr-2"></i> 1. Live QR Ingestion
        </button>
        <button onclick="switchHospitalTab('bookings')" id="hospTabBookings" class="px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-calendar-check mr-2"></i> 2. Patient Bookings (<span id="hospitalBookingsBadge">0</span>)
        </button>
        <button onclick="switchHospitalTab('logs')" id="hospTabLogs" class="px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-table-list mr-2"></i> 3. OPD Logs (<span id="totalLogsBadge">1</span>)
        </button>
        <button onclick="switchHospitalTab('summary')" id="hospTabSummary" class="px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-print mr-2"></i> 4. Print Summary
        </button>
        <button onclick="switchHospitalTab('doctors')" id="hospTabDoctors" class="px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl">
          <i class="fa-solid fa-user-doctor mr-2"></i> 5. Doctor Roster (<span id="rosterCountBadge">0</span>)
        </button>
      </div>

      <!-- Hospital Tab 1: Live QR Scanner & Immediate Diagnostic Upload -->
      <div id="hospPageIngestion" class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        <!-- Live Scanner & Patient Intake (2 Cols) -->
        <div class="lg:col-span-2 space-y-6">
          
          <!-- Live QR Scanner Card -->
          <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
            <div class="flex flex-col md:flex-row justify-between md:items-center gap-3">
              <div>
                <h3 class="font-bold text-slate-800 text-sm md:text-base flex items-center">
                  <i class="fa-solid fa-camera text-teal-600 mr-2"></i> Live Camera & Image File QR Scanner
                </h3>
                <p class="text-xs text-slate-500">Scan via video stream or upload a QR image file for instant patient triage.</p>
              </div>
              <div class="flex flex-wrap gap-2">
                <button onclick="toggleLiveCameraScanner()" id="btnToggleCamera" class="bg-teal-700 hover:bg-teal-800 text-white font-semibold px-3 py-1.5 rounded-xl text-xs flex items-center space-x-1 transition">
                  <i class="fa-solid fa-video"></i> <span id="cameraBtnLabel">Start Camera Scanner</span>
                </button>
                <label class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold px-3 py-1.5 rounded-xl text-xs flex items-center space-x-1 cursor-pointer transition">
                  <i class="fa-solid fa-file-image"></i> <span>Upload QR Image</span>
                  <input type="file" id="qrImageFileInput" accept="image/*" onchange="handleQRImageUpload(event)" class="hidden" />
                </label>
              </div>
            </div>

            <!-- Video Scanner Container -->
            <div id="cameraScannerBox" class="hidden border-2 border-teal-500 rounded-2xl overflow-hidden bg-black p-2">
              <div id="qr-reader" style="width: 100%;"></div>
            </div>

            <!-- Hidden container for scanning image files -->
            <div id="qr-file-reader" class="hidden"></div>

            <!-- Fallback Simulation Button -->
            <div class="p-3 bg-slate-50 border rounded-xl flex items-center justify-between">
              <span class="text-xs text-slate-600">Quick Simulation using active patient profile:</span>
              <button onclick="simulateScanPatient()" class="bg-slate-800 hover:bg-slate-900 text-white px-3 py-1.5 rounded-lg text-xs font-bold transition">
                Simulate QR Scan
              </button>
            </div>

            <!-- Patient Ingestion Details Card (Populated by Scanner) -->
            <div id="scannedPatientCard" class="border-2 border-teal-600 rounded-2xl p-5 bg-teal-50/40 hidden space-y-4">
              <div class="flex justify-between items-start border-b border-teal-200 pb-3">
                <div class="flex items-center space-x-3">
                  <img id="scannedPatientAvatar" src="https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=100&auto=format&fit=crop&q=80" class="w-12 h-12 rounded-full object-cover border-2 border-teal-600" />
                  <div>
                    <div class="flex items-center space-x-2">
                      <h4 class="font-extrabold text-teal-950 text-base" id="scanName">-</h4>
                      <span class="text-[10px] font-extrabold bg-teal-200 text-teal-900 px-2 py-0.5 rounded-full" id="scanVisitBadge">Visit #1</span>
                    </div>
                    <p class="text-xs text-slate-600" id="scanMeta">-</p>
                  </div>
                </div>
                <span class="text-xs bg-emerald-100 text-emerald-800 font-bold px-2.5 py-1 rounded-lg">Verified Intake</span>
              </div>

              <!-- Case-taking data -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-3 text-xs">
                <div class="bg-white p-2.5 rounded-xl border border-teal-100">
                  <p class="text-[11px] font-bold text-slate-500">Disease / Reason for Visit:</p>
                  <p class="font-semibold text-slate-800" id="scanComplaints">-</p>
                </div>
                <div class="bg-white p-2.5 rounded-xl border border-teal-100">
                  <p class="text-[11px] font-bold text-slate-500">Blood Group & Age:</p>
                  <p class="font-semibold text-slate-800" id="scanBloodAge">-</p>
                </div>
              </div>

              <!-- OPD Ingestion Form -->
              <div class="grid grid-cols-1 md:grid-cols-2 gap-3 pt-2">
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Doctor Treated / Assigned Specialist *</label>
                  <select id="logDoctorSelect" class="w-full p-2 border rounded-xl text-xs bg-white font-medium outline-none"></select>
                </div>
                <div>
                  <label class="block text-[11px] font-bold text-slate-700 mb-1">Confirmed Disease / Clinical Diagnosis *</label>
                  <input type="text" id="logPurpose" value="Chronic Lumbar Spondylosis with Lower Back Nerve Compression" class="w-full p-2 border rounded-xl text-xs bg-white outline-none" />
                </div>
              </div>

              <div class="flex flex-wrap gap-2 pt-1">
                <button onclick="commitVisitLog()" class="flex-1 bg-teal-800 hover:bg-teal-900 text-white py-2.5 rounded-xl text-xs font-bold shadow transition flex items-center justify-center space-x-2">
                  <i class="fa-solid fa-check-double"></i> <span>Check-in Patient & Create Log</span>
                </button>
                <button onclick="goToHospitalSummaryPrint()" class="bg-slate-800 hover:bg-slate-900 text-white px-4 py-2.5 rounded-xl text-xs font-bold transition flex items-center space-x-1.5">
                  <i class="fa-solid fa-print"></i> <span>Print Medical Summary</span>
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Direct Diagnostic File Upload Column (1 Col) -->
        <div class="space-y-6">
          <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
            <div>
              <h3 class="font-bold text-slate-800 text-sm flex items-center">
                <i class="fa-solid fa-cloud-arrow-up text-blue-600 mr-2"></i> Upload Patient Diagnostic Report
              </h3>
              <p class="text-xs text-slate-500">Upload Blood tests, X-Rays, or MRI scans directly to patient vaults.</p>
            </div>
            
            <form onsubmit="handleHospitalReportUpload(event)" class="space-y-3">
              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Patient Name / Policy</label>
                <input type="text" id="repPatientName" required value="Rahul Sharma" class="w-full p-2 border rounded-xl text-xs bg-slate-50 outline-none" />
              </div>

              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Report Category</label>
                <select id="repCategory" class="w-full p-2 border rounded-xl text-xs bg-slate-50 outline-none font-medium">
                  <option value="Blood Test Report (CBC)">Complete Blood Count (CBC)</option>
                  <option value="X-Ray Scan (Lumbar Spine)">X-Ray Scan</option>
                  <option value="MRI Brain/Spine">MRI Scan</option>
                  <option value="Pathology Report">Pathology Report</option>
                  <option value="Prakriti Diagnostic Assessment">Ayurvedic Prakriti Analysis</option>
                </select>
              </div>

              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Authorizing Diagnostic Specialist</label>
                <input type="text" id="repDoctor" value="Dr. Ananya Roy (Radiologist)" class="w-full p-2 border rounded-xl text-xs bg-slate-50 outline-none" />
              </div>

              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Select File (Image / PDF)</label>
                <input type="file" id="repFileInput" required accept="image/*,application/pdf" onchange="previewUploadFile(event)" class="w-full text-xs text-slate-500 file:mr-2 file:py-1.5 file:px-3 file:rounded-lg file:border-0 file:text-xs file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100 border p-1 rounded-xl" />
              </div>

              <div id="filePreviewNote" class="text-[11px] text-blue-700 font-semibold hidden"></div>

              <button type="submit" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2.5 rounded-xl text-xs shadow transition flex items-center justify-center space-x-2">
                <i class="fa-solid fa-file-arrow-up"></i> <span>Direct Upload to Patient</span>
              </button>
            </form>
          </div>
        </div>
      </div>

      <!-- Hospital Tab 2: Dedicated Patient Bookings, Status Triage & File Attachment -->
      <div id="hospPageBookings" class="space-y-4 hidden">
        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
          <div class="flex flex-col md:flex-row justify-between md:items-center gap-3">
            <div>
              <h2 class="font-bold text-slate-800 text-base">Hospital Patient OPD Bookings & Triage</h2>
              <p class="text-xs text-slate-500">Live booking queue for your hospital. Mark treated status, attach reports, or flag for revisitation.</p>
            </div>
            <div class="flex gap-2">
              <input type="text" id="bookingSearchInput" onkeyup="filterHospitalBookings()" placeholder="Search patient or doctor..." class="p-2 border rounded-xl text-xs bg-slate-50 outline-none" />
            </div>
          </div>

          <!-- Bookings Table -->
          <div class="overflow-x-auto rounded-xl border border-slate-200">
            <table class="w-full text-left text-xs text-slate-600">
              <thead class="bg-slate-100 text-slate-800 uppercase text-[11px] font-bold">
                <tr>
                  <th class="p-3">Slot / Date</th>
                  <th class="p-3">Patient Details</th>
                  <th class="p-3">Doctor Assigned</th>
                  <th class="p-3">Specialty</th>
                  <th class="p-3 text-center">Treatment Status</th>
                  <th class="p-3 text-center">Follow-up</th>
                  <th class="p-3 text-center">Actions</th>
                </tr>
              </thead>
              <tbody id="hospitalBookingsBody" class="divide-y divide-slate-100"></tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Hospital Tab 3: Dedicated Separate OPD Patient Logs Vault -->
      <div id="hospPageLogs" class="space-y-4 hidden">
        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
          <div class="flex flex-col md:flex-row justify-between md:items-center gap-3">
            <div>
              <h2 class="font-bold text-slate-800 text-base">OPD Central Patient Log Vault</h2>
              <p class="text-xs text-slate-500">Complete record of scanned check-ins, assigned specialists, and visit frequencies.</p>
            </div>
            <div class="flex gap-2">
              <input type="text" id="logSearchInput" onkeyup="filterHospitalLogs()" placeholder="Search patient name..." class="p-2 border rounded-xl text-xs bg-slate-50 outline-none" />
              <button onclick="exportLogsCSV()" class="bg-slate-800 hover:bg-slate-900 text-white px-3 py-2 rounded-xl text-xs font-bold transition">
                <i class="fa-solid fa-file-csv mr-1"></i> Export Logs
              </button>
            </div>
          </div>

          <!-- Logs Table -->
          <div class="overflow-x-auto rounded-xl border border-slate-200">
            <table class="w-full text-left text-xs text-slate-600">
              <thead class="bg-slate-100 text-slate-800 uppercase text-[11px] font-bold">
                <tr>
                  <th class="p-3">Time</th>
                  <th class="p-3">Patient Name</th>
                  <th class="p-3">Visit No.</th>
                  <th class="p-3">Doctor Treated</th>
                  <th class="p-3">Disease / Diagnosis</th>
                  <th class="p-3 text-center">Actions</th>
                </tr>
              </thead>
              <tbody id="hospitalLogBody" class="divide-y divide-slate-100"></tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Hospital Tab 4: Dedicated Print Medical Summary Page -->
      <div id="hospPageSummary" class="space-y-4 hidden">
        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-6">
          <div class="flex justify-between items-center border-b pb-4">
            <div>
              <h2 class="font-bold text-slate-800 text-base">Official Clinical Summary Sheet</h2>
              <p class="text-xs text-slate-500">Strictly focused on the hospital header, core patient metrics, treating doctor, and disease.</p>
            </div>
            <button onclick="window.print()" class="bg-teal-700 hover:bg-teal-800 text-white font-bold px-5 py-2.5 rounded-xl text-xs shadow flex items-center space-x-2">
              <i class="fa-solid fa-print"></i> <span>Print Clinical Summary</span>
            </button>
          </div>

          <!-- Printable Document Container -->
          <div id="printReportSection" class="bg-white p-8 border-2 border-slate-300 rounded-2xl space-y-6 text-slate-800 max-w-4xl mx-auto shadow-sm">
            
            <!-- 1. UPPER SECTION: Name and Photo of Hospital -->
            <div class="flex justify-between items-center border-b-2 border-slate-800 pb-5">
              <div class="flex items-center space-x-4">
                <img id="printHospitalLogo" src="https://images.unsplash.com/photo-1519494026892-80bbd2d6fd0d?w=120&auto=format&fit=crop&q=80" alt="Hospital Photo" class="w-16 h-16 rounded-xl object-cover border border-slate-300 shadow-sm" />
                <div>
                  <h1 class="text-2xl font-black uppercase text-[#1e4d79] tracking-tight" id="printHospitalTitle">ST. JAMES HOSPITAL</h1>
                  <p class="text-xs font-bold text-slate-600">CLINICAL OPD MEDICAL SUMMARY REPORT</p>
                  <p class="text-[11px] text-slate-400">Department of Ayush & Clinical Medicine</p>
                </div>
              </div>
              <div class="text-right text-xs space-y-0.5">
                <p><strong>Date:</strong> <span id="printCurrentDate"></span></p>
                <p><strong>Record Status:</strong> <span class="bg-emerald-100 text-emerald-800 px-2 py-0.5 rounded font-bold">Verified Intake</span></p>
              </div>
            </div>

            <!-- 2. NEXT SECTION: Patient Name, Blood Group, Age, and Phone Number -->
            <div class="bg-slate-50 p-5 rounded-xl border border-slate-200">
              <h3 class="text-xs font-black uppercase tracking-wider text-teal-900 border-b pb-2 mb-3">
                <i class="fa-solid fa-user-injured mr-1.5"></i> Patient Identification & Vitals
              </h3>
              <div class="grid grid-cols-2 md:grid-cols-4 gap-4 text-xs">
                <div>
                  <span class="text-slate-500 font-medium block">Patient Name:</span>
                  <span class="font-extrabold text-sm text-slate-900" id="printName">Rahul Sharma</span>
                </div>
                <div>
                  <span class="text-slate-500 font-medium block">Blood Group:</span>
                  <span class="font-black text-rose-700 text-sm" id="printBloodGroup">O+</span>
                </div>
                <div>
                  <span class="text-slate-500 font-medium block">Age:</span>
                  <span class="font-extrabold text-sm text-slate-900" id="printAge">32 Years</span>
                </div>
                <div>
                  <span class="text-slate-500 font-medium block">Phone Number:</span>
                  <span class="font-extrabold text-sm text-slate-900 font-mono" id="printPhone">+91 98765 43210</span>
                </div>
              </div>
            </div>

            <!-- 3. CLINICAL SUMMARY: Doctor Treated and What is the Disease -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 text-xs">
              <div class="bg-teal-50/50 p-4 rounded-xl border border-teal-200">
                <span class="text-teal-900 font-bold uppercase text-[11px] block mb-1">
                  <i class="fa-solid fa-user-doctor mr-1"></i> Doctor Treated:
                </span>
                <p class="text-base font-extrabold text-teal-950" id="printDoctorTreated">Dr. Vikram Sethi (Kayachikitsa / Internal Med)</p>
              </div>

              <div class="bg-amber-50/50 p-4 rounded-xl border border-amber-200">
                <span class="text-amber-900 font-bold uppercase text-[11px] block mb-1">
                  <i class="fa-solid fa-virus-covid mr-1"></i> What is the Disease / Diagnosis:
                </span>
                <p class="text-base font-extrabold text-amber-950" id="printDisease">Chronic Lumbar Spondylosis with Lower Back Nerve Compression</p>
              </div>
            </div>

            <!-- Signatures Section -->
            <div class="pt-10 grid grid-cols-2 gap-8 text-xs text-center">
              <div class="border-t border-slate-400 pt-2">
                <p class="font-bold text-slate-800">Patient / Attendant Signature</p>
              </div>
              <div class="border-t border-slate-400 pt-2">
                <p class="font-bold text-slate-800">Treating Doctor Signature & Stamp</p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Hospital Tab 5: Doctor Roster & Photo Registration & Delete Option -->
      <div id="hospPageDoctors" class="grid grid-cols-1 lg:grid-cols-3 gap-6 hidden">
        
        <!-- Registration Form (1 Col) -->
        <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
          <div>
            <h3 class="font-bold text-slate-800 text-sm flex items-center">
              <i class="fa-solid fa-user-plus text-teal-600 mr-2"></i> Register New Doctor
            </h3>
            <p class="text-xs text-slate-500">Add physicians under your hospital roster with full specialty classification.</p>
          </div>

          <form onsubmit="addNewDoctor(event)" class="space-y-3">
            <div>
              <label class="block text-[11px] font-bold text-slate-700 mb-1">Doctor Name *</label>
              <input type="text" id="docName" required placeholder="Dr. Rajesh Varma" class="w-full p-2 border rounded-xl text-xs outline-none" />
            </div>
            <div>
              <label class="block text-[11px] font-bold text-slate-700 mb-1">Hospital Affiliation *</label>
              <input type="text" id="docHospitalName" required placeholder="Hospital Name" class="w-full p-2 border rounded-xl text-xs outline-none font-medium bg-slate-50" />
            </div>
            <div>
              <label class="block text-[11px] font-bold text-slate-700 mb-1">Department / Specialty *</label>
              <select id="docDept" class="w-full p-2 border rounded-xl text-xs outline-none font-medium">
                <option value="General Practice">General Practice</option>
                <option value="Gastroenterology">Gastroenterology</option>
                <option value="Cardiology">Cardiology</option>
                <option value="Pediatrics">Pediatrics</option>
                <option value="Endocrinology">Endocrinology</option>
                <option value="Orthopedic">Orthopedic</option>
                <option value="Pathology">Pathology</option>
                <option value="Radiology">Radiology</option>
                <option value="Psychiatry">Psychiatry</option>
                <option value="Ophthalmology">Ophthalmology</option>
                <option value="Neurology">Neurology</option>
                <option value="Pulmonology">Pulmonology</option>
                <option value="Otolaryngology (ENT)">Otolaryngology (ENT)</option>
                <option value="Urology">Urology</option>
                <option value="Kayachikitsa (Internal Med)">Kayachikitsa (Ayurveda)</option>
                <option value="Panchakarma">Panchakarma</option>
              </select>
            </div>
            <div class="grid grid-cols-2 gap-2">
              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Status</label>
                <select id="docStatus" class="w-full p-2 border rounded-xl text-xs outline-none font-bold">
                  <option value="Available" class="text-emerald-600">Available</option>
                  <option value="On Leave" class="text-rose-600">On Leave</option>
                </select>
              </div>
              <div>
                <label class="block text-[11px] font-bold text-slate-700 mb-1">Room / OPD No.</label>
                <input type="text" id="docRoom" placeholder="Chamber 204" class="w-full p-2 border rounded-xl text-xs outline-none" />
              </div>
            </div>
            <div>
              <label class="block text-[11px] font-bold text-slate-700 mb-1">Doctor Photo (Upload)</label>
              <input type="file" id="docPhotoInput" accept="image/*" class="w-full text-xs text-slate-500 file:mr-2 file:py-1 file:px-2 file:rounded-lg file:border-0 file:text-xs file:bg-slate-100 border p-1 rounded-xl" />
            </div>

            <button type="submit" class="w-full bg-teal-700 hover:bg-teal-800 text-white font-bold py-2.5 rounded-xl text-xs shadow transition">
              + Register Doctor Profile
            </button>
          </form>
        </div>

        <!-- Hospital Doctor Roster List (2 Cols with Delete Action) -->
        <div class="lg:col-span-2 bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
          <div class="flex justify-between items-center">
            <div>
              <h3 class="font-bold text-slate-800 text-sm">Manage Hospital Doctor Roster</h3>
              <p class="text-xs text-slate-500">Toggle availability or delete doctors who have left the institution.</p>
            </div>
          </div>
          <div id="hospitalDoctorRosterList" class="grid grid-cols-1 md:grid-cols-2 gap-3 text-xs"></div>
        </div>
      </div>

    </div>
  </main>

  <!-- APPOINTMENT MODAL -->
  <div id="bookingModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm hidden flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-3xl max-w-md w-full p-6 space-y-4 shadow-2xl border border-slate-100">
      <div class="flex justify-between items-center border-b pb-3">
        <h3 class="font-black text-slate-800 text-base">Book Specialist OPD Appointment</h3>
        <button onclick="closeBookingModal()" class="text-slate-400 hover:text-slate-600 text-xl font-bold">&times;</button>
      </div>
      
      <!-- Doctor and Hospital Info Container in Modal -->
      <div id="modalDoctorDetails" class="text-xs bg-teal-50/60 p-4 rounded-2xl border border-teal-100 space-y-2"></div>
      
      <div>
        <label class="block text-xs font-bold text-slate-700 mb-1">Appointment Date & Slot</label>
        <input type="date" id="bookDate" class="w-full p-2.5 border rounded-xl text-xs mb-2 bg-slate-50 outline-none" />
        <select id="bookSlot" class="w-full p-2.5 border rounded-xl text-xs bg-slate-50 outline-none font-medium">
          <option>09:30 AM - 10:00 AM (Morning OPD)</option>
          <option>10:30 AM - 11:00 AM (Morning OPD)</option>
          <option>02:00 PM - 02:30 PM (Afternoon OPD)</option>
          <option>03:30 PM - 04:00 PM (Afternoon OPD)</option>
        </select>
      </div>
      <button onclick="confirmAppointment()" class="w-full bg-teal-700 hover:bg-teal-800 text-white font-bold py-3 rounded-xl text-xs shadow-lg transition">
        Confirm Booking
      </button>
    </div>
  </div>

  <!-- QUICK REPORT UPLOAD MODAL FOR LOG VAULT & BOOKINGS -->
  <div id="quickUploadModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm hidden flex items-center justify-center p-4 z-50">
    <div class="bg-white rounded-3xl max-w-md w-full p-6 space-y-4 shadow-2xl border border-slate-100">
      <div class="flex justify-between items-center border-b pb-3">
        <h3 class="font-black text-slate-800 text-base">Attach Diagnostic Report to Patient</h3>
        <button onclick="closeQuickUploadModal()" class="text-slate-400 hover:text-slate-600 text-xl font-bold">&times;</button>
      </div>
      <div class="text-xs space-y-3">
        <p><strong>Patient:</strong> <span id="quickUploadPatientName" class="text-teal-800 font-bold"></span></p>
        <div>
          <label class="block text-[11px] font-bold text-slate-700 mb-1">Report Category</label>
          <select id="quickRepCategory" class="w-full p-2 border rounded-xl text-xs bg-slate-50">
            <option value="Blood Test Report (CBC)">Complete Blood Count (CBC)</option>
            <option value="X-Ray Scan (Lumbar Spine)">X-Ray Scan</option>
            <option value="MRI Brain/Spine">MRI Scan</option>
            <option value="Pathology Report">Pathology Report</option>
            <option value="Prakriti Diagnostic Assessment">Ayurvedic Prakriti Analysis</option>
          </select>
        </div>
        <div>
          <label class="block text-[11px] font-bold text-slate-700 mb-1">Select File</label>
          <input type="file" id="quickFileInput" accept="image/*,application/pdf" class="w-full text-xs text-slate-500 border p-1 rounded-xl" />
        </div>
        <button onclick="saveQuickReport()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2.5 rounded-xl text-xs shadow">
          Upload Directly to Vault
        </button>
      </div>
    </div>
  </div>

  <!-- JAVASCRIPT APPLICATION LOGIC WITH FIREBASE SYNC & FIXED USER ISOLATION -->
  <script>
    // ================= FIREBASE CLOUD CONFIGURATION =================
    const firebaseConfig = {
      apiKey: "AIzaSyA7Jx2WHDSHdq3Q0XU8QIvV-k-wwO4wC2s",
      authDomain: "aayush-971b7.firebaseapp.com",
      databaseURL: "https://aayush-971b7-default-rtdb.firebaseio.com",
      projectId: "aayush-971b7",
      storageBucket: "aayush-971b7.firebasestorage.app",
      messagingSenderId: "354997155318",
      appId: "1:354997155318:web:32e5752f31724016b003d7"
    };

    // Initialize Firebase
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    // System State
    let currentHospitalName = "St. James Hospital";
    let hospitalLogoUrl = "https://images.unsplash.com/photo-1519494026892-80bbd2d6fd0d?w=120&auto=format&fit=crop&q=80";
    let activeAuthRole = "patient";
    let activeAuthTab = "login";
    let currentLoggedUser = null;
    let patientAvatarUrl = "https://images.unsplash.com/photo-1535713875002-d1d0cf377fde?w=100&auto=format&fit=crop&q=80";

    // Cloud Registered Users Database
    let registeredUsers = [
      { email: "rahul.sharma@medmail.com", pass: "123456", name: "Rahul Sharma", role: "patient" },
      { email: "staff@stjames.com", pass: "staff123", name: "Dr. Admin", role: "hospital", hospital: "St. James Hospital" }
    ];

    let doctors = [];
    let patientBookings = [];
    let patientReports = [];
    let visitLogs = [];

    let html5QrCodeScanner = null;
    let isCameraRunning = false;
    let currentSelectedDoctor = null;
    let uploadedFileBlobUrl = null;
    let patientVisitCount = 1;
    let quickUploadTargetPatient = "";

    // ================= REAL-TIME FIREBASE LISTENERS =================
    db.ref('users').on('value', (snapshot) => {
      const data = snapshot.val();
      if (data) {
        registeredUsers = Object.values(data);
      } else {
        db.ref('users').set(registeredUsers);
      }
    });

    db.ref('doctors').on('value', (snapshot) => {
      const data = snapshot.val();
      if (data) {
        doctors = Object.values(data);
      } else {
        doctors = [
          { id: 1, name: "Dr. Vikram Sethi", hospital: "St. James Hospital", dept: "Kayachikitsa (Internal Med)", status: "Available", room: "Room 101", photo: "https://images.unsplash.com/photo-1622253692010-333f2da6031d?w=150&auto=format&fit=crop&q=80" },
          { id: 2, name: "Dr. Ananya Roy", hospital: "Apollo Apex Care", dept: "Radiology", status: "Available", room: "Scan Lab 2", photo: "https://images.unsplash.com/photo-1594824813583-3760a92b9576?w=150&auto=format&fit=crop&q=80" }
        ];
        db.ref('doctors').set(doctors);
      }
      renderPatientDoctors();
      renderHospitalRoster();
      populateDoctorSelect();
      populateHospitalFilterDropdown();
    });

    db.ref('bookings').on('value', (snapshot) => {
      const data = snapshot.val();
      if (data) {
        patientBookings = Object.values(data);
      } else {
        patientBookings = [];
      }
      renderHospitalBookings();
      renderPatientAppointments();
    });

    db.ref('reports').on('value', (snapshot) => {
      const data = snapshot.val();
      if (data) {
        patientReports = Object.values(data);
      } else {
        patientReports = [];
      }
      renderPatientReports();
    });

    db.ref('logs').on('value', (snapshot) => {
      const data = snapshot.val();
      if (data) {
        visitLogs = Object.values(data);
      } else {
        visitLogs = [];
      }
      renderHospitalLogs();
    });

    // ================= AUTHENTICATION LOGIC =================
    function setAuthTab(tab) {
      activeAuthTab = tab;
      hideAuthAlert();
      const btnLogin = document.getElementById('tabBtnLogin');
      const btnSignup = document.getElementById('tabBtnSignup');
      const signupNameGroup = document.getElementById('signupNameGroup');
      const submitBtn = document.getElementById('authSubmitBtn');

      if (tab === 'login') {
        btnLogin.className = "flex-1 pb-2.5 text-xs font-bold text-teal-700 border-b-2 border-teal-600 transition";
        btnSignup.className = "flex-1 pb-2.5 text-xs font-semibold text-slate-400 hover:text-slate-700 transition";
        signupNameGroup.classList.add('hidden');
        submitBtn.innerHTML = `<span>Sign In</span> <i class="fa-solid fa-arrow-right text-[10px]"></i>`;
      } else {
        btnSignup.className = "flex-1 pb-2.5 text-xs font-bold text-teal-700 border-b-2 border-teal-600 transition";
        btnLogin.className = "flex-1 pb-2.5 text-xs font-semibold text-slate-400 hover:text-slate-700 transition";
        signupNameGroup.classList.remove('hidden');
        submitBtn.innerHTML = `<span>Create Account</span> <i class="fa-solid fa-user-plus text-[10px]"></i>`;
      }
    }

    function setAuthRole(role) {
      activeAuthRole = role;
      hideAuthAlert();
      const btnPat = document.getElementById('authRolePatient');
      const btnHosp = document.getElementById('authRoleHospital');
      const hospGroup = document.getElementById('hospitalAuthGroup');
      const idLabel = document.getElementById('authIdentifierLabel');
      const nameLabel = document.getElementById('nameFieldLabel');

      if (role === 'patient') {
        btnPat.className = "py-2.5 rounded-lg bg-white shadow-sm text-teal-800 font-bold transition";
        btnHosp.className = "py-2.5 rounded-lg text-slate-500 hover:text-slate-800 transition";
        hospGroup.classList.add('hidden');
        idLabel.textContent = "Patient Email / ABHA ID *";
        nameLabel.textContent = "Patient Full Name *";
        document.getElementById('authIdentifier').placeholder = "Enter email or ABHA ID";
      } else {
        btnHosp.className = "py-2.5 rounded-lg bg-white shadow-sm text-amber-700 font-bold transition";
        btnPat.className = "py-2.5 rounded-lg text-slate-500 hover:text-slate-800 transition";
        hospGroup.classList.remove('hidden');
        idLabel.textContent = "Staff Username / Work Email *";
        nameLabel.textContent = "Staff Member Name *";
        document.getElementById('authIdentifier').placeholder = "Enter staff ID / work email";
      }
    }

    function showAuthAlert(message, type = "error") {
      const box = document.getElementById('authAlertBox');
      const txt = document.getElementById('authAlertText');
      box.className = type === "error" 
        ? "p-3 rounded-xl text-xs flex items-center space-x-2 bg-rose-50 border border-rose-200 text-rose-800" 
        : "p-3 rounded-xl text-xs flex items-center space-x-2 bg-teal-50 border border-teal-200 text-teal-800";
      txt.innerHTML = message;
      box.classList.remove('hidden');
    }

    function hideAuthAlert() {
      document.getElementById('authAlertBox').classList.add('hidden');
    }

    function previewHospitalLogo(e) {
      if (e.target.files && e.target.files[0]) {
        hospitalLogoUrl = URL.createObjectURL(e.target.files[0]);
      }
    }

    function handleAuthSubmit(e) {
      e.preventDefault();
      hideAuthAlert();
      const identifier = document.getElementById('authIdentifier').value.trim().toLowerCase();
      const password = document.getElementById('authPassword').value;

      if (activeAuthRole === 'hospital') {
        const hospName = document.getElementById('authHospitalName').value.trim();
        if (hospName) currentHospitalName = hospName;
      }

      if (activeAuthTab === 'login') {
        const existing = registeredUsers.find(u => 
          u.email.toLowerCase() === identifier && u.role === activeAuthRole
        );

        if (!existing) {
          showAuthAlert(`No account found for "<strong>${identifier}</strong>". Please switch to <strong>Create New Account</strong>.`);
          return;
        }

        if (existing.pass !== password) {
          showAuthAlert("Invalid password. Please enter the correct password.");
          return;
        }

        currentLoggedUser = existing;
        if (existing.hospital) currentHospitalName = existing.hospital;
        launchPortal(existing);

      } else {
        const fullName = document.getElementById('authFullName').value.trim();
        if (!fullName) {
          showAuthAlert("Please provide your full name to create an account.");
          return;
        }

        const duplicate = registeredUsers.find(u => 
          u.email.toLowerCase() === identifier && u.role === activeAuthRole
        );

        if (duplicate) {
          showAuthAlert(`An account with "<strong>${identifier}</strong>" already exists. Please Sign In.`);
          return;
        }

        const newUser = {
          email: identifier,
          pass: password,
          name: fullName,
          role: activeAuthRole,
          hospital: activeAuthRole === 'hospital' ? currentHospitalName : null
        };

        const userKey = 'user_' + Date.now();
        db.ref('users/' + userKey).set(newUser).then(() => {
          registeredUsers.push(newUser);
          currentLoggedUser = newUser;
          launchPortal(newUser);
        }).catch(err => {
          showAuthAlert("Error creating account in cloud database: " + err.message);
        });
      }
    }

    function launchPortal(user) {
      const pSec = document.getElementById('patientSection');
      const hSec = document.getElementById('hospitalSection');

      document.getElementById('userGreetingName').textContent = user.name;
      document.getElementById('userGreetingRole').textContent = user.role === 'patient' ? 'Patient' : `${currentHospitalName} (Staff)`;

      if (user.role === 'patient') {
        pSec.classList.remove('hidden');
        hSec.classList.add('hidden');
        document.getElementById('headerMainTitle').textContent = "AyushCare Patient Portal";
        document.getElementById('headerRoleBadge').textContent = "Patient Mode";
        document.getElementById('headerSubtitle').textContent = "Medical Registration, Touchless QR & Specialist Booking";
        document.getElementById('pFullName').value = user.name;
        document.getElementById('pEmail').value = user.email;
        updatePatientDisplayName();
        renderPatientDoctors();
        renderPatientReports();
        renderPatientAppointments();
      } else {
        pSec.classList.add('hidden');
        hSec.classList.remove('hidden');
        document.getElementById('headerMainTitle').textContent = `${currentHospitalName} OPD`;
        document.getElementById('headerRoleBadge').textContent = "Hospital Desk";
        document.getElementById('headerSubtitle').textContent = "Clinical Triage, Patient Queue & Diagnostic Records";
        updateHospitalNameDisplays();
        renderHospitalBookings();
        renderHospitalRoster();
        renderHospitalLogs();
        populateDoctorSelect();
      }

      document.getElementById('authScreen').classList.add('hidden');
    }

    function logoutSession() {
      currentLoggedUser = null;
      hideAuthAlert();
      document.getElementById('patientSection').classList.add('hidden');
      document.getElementById('hospitalSection').classList.add('hidden');
      document.getElementById('authScreen').classList.remove('hidden');
    }

    function updateHospitalNameDisplays() {
      document.getElementById('printHospitalTitle').textContent = currentHospitalName.toUpperCase();
      document.getElementById('hospitalModeBannerTitle').textContent = `${currentHospitalName.toUpperCase()} OPD DESK`;
      document.getElementById('hospitalBannerLogo').src = hospitalLogoUrl;
      document.getElementById('printHospitalLogo').src = hospitalLogoUrl;
      document.getElementById('docHospitalName').value = currentHospitalName;
      populateHospitalFilterDropdown();
    }

    function populateHospitalFilterDropdown() {
      const filterHosp = document.getElementById('filterHospital');
      if (!filterHosp) return;
      const currentVal = filterHosp.value;
      const uniqueHospitals = Array.from(new Set(doctors.map(d => d.hospital)));
      
      filterHosp.innerHTML = '<option value="All">All Hospitals</option>';
      uniqueHospitals.forEach(h => {
        const opt = document.createElement('option');
        opt.value = h;
        opt.textContent = h;
        filterHosp.appendChild(opt);
      });
      if (uniqueHospitals.includes(currentVal)) {
        filterHosp.value = currentVal;
      }
    }

    // ================= PATIENT TABS & ISOLATED DATA RENDERING =================
    function switchPatientTab(tabNum) {
      [1, 2, 3, 4].forEach(num => {
        const page = document.getElementById(`ptPage${num}`);
        if (page) page.classList.toggle('hidden', num !== tabNum);
        const tabBtn = document.getElementById(`ptTab${num}`);
        if (tabBtn) {
          if (num === tabNum) {
            tabBtn.className = "tab-active-patient px-5 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl";
          } else {
            tabBtn.className = "px-5 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl";
          }
        }
      });
      if (tabNum === 2) renderPatientDoctors();
      if (tabNum === 3) renderPatientReports();
      if (tabNum === 4) renderPatientAppointments();
    }

    function renderPatientAppointments() {
      const tbody = document.getElementById('patientAppointmentsBody');
      const badge = document.getElementById('patientBookingCountBadge');
      if (!tbody || !currentLoggedUser) return;
      tbody.innerHTML = '';

      // ISOLATION FIX: Show ONLY bookings belonging to this specific logged-in patient email/name
      const userBookings = patientBookings.filter(b => 
        (b.patientEmail && b.patientEmail.toLowerCase() === currentLoggedUser.email.toLowerCase()) ||
        (b.patientName && b.patientName.toLowerCase() === currentLoggedUser.name.toLowerCase())
      );

      badge.textContent = userBookings.length;

      if (userBookings.length === 0) {
        tbody.innerHTML = `<tr><td colspan="4" class="p-6 text-center text-slate-400">You have 0 active bookings. Go to '2. Book Specialist Doctor' to schedule one.</td></tr>`;
        return;
      }

      userBookings.forEach(b => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td class="p-3">
            <span class="font-bold text-slate-900 block">${b.hospital}</span>
            <span class="text-[10px] text-teal-700 font-semibold">${b.dept}</span>
          </td>
          <td class="p-3 font-semibold text-slate-900">${b.doctorName}</td>
          <td class="p-3">
            <span class="font-bold text-slate-800 block">${b.date}</span>
            <span class="text-[10px] text-slate-400 font-mono">${b.slot}</span>
          </td>
          <td class="p-3 text-center">
            <span class="px-2.5 py-1 rounded-full text-[10px] font-bold ${b.treatedStatus === 'Treated' ? 'bg-emerald-100 text-emerald-800' : 'bg-amber-100 text-amber-800'}">
              ${b.treatedStatus}
            </span>
          </td>
        `;
        tbody.appendChild(tr);
      });
    }

    function renderPatientReports() {
      const container = document.getElementById('patientReportsContainer');
      const badge = document.getElementById('reportCountBadge');
      if (!container || !currentLoggedUser) return;
      container.innerHTML = '';

      // ISOLATION FIX: Show ONLY reports uploaded for this specific user
      const userReports = patientReports.filter(r => 
        (r.patientEmail && r.patientEmail.toLowerCase() === currentLoggedUser.email.toLowerCase()) ||
        (r.patientName && r.patientName.toLowerCase() === currentLoggedUser.name.toLowerCase())
      );

      badge.textContent = userReports.length;

      if (userReports.length === 0) {
        container.innerHTML = `<p class="text-xs text-slate-400 col-span-2">You have 0 medical reports uploaded by hospitals yet.</p>`;
        return;
      }

      userReports.forEach(rep => {
        const item = document.createElement('div');
        item.className = "p-4 border rounded-2xl bg-slate-50 hover:bg-white transition flex flex-col justify-between space-y-3";
        item.innerHTML = `
          <div>
            <div class="flex justify-between items-start">
              <span class="bg-blue-100 text-blue-800 text-[10px] font-bold px-2 py-0.5 rounded-full">${rep.category}</span>
              <span class="text-xs text-slate-400">${rep.date}</span>
            </div>
            <h4 class="font-bold text-sm text-slate-800 mt-2">${rep.title}</h4>
            <p class="text-xs text-slate-500">Ordered / Verified by: ${rep.doctor}</p>
          </div>
          <div class="flex items-center gap-2 pt-2 border-t">
            <a href="${rep.fileUrl}" target="_blank" class="flex-1 text-center bg-teal-700 hover:bg-teal-800 text-white py-1.5 rounded-xl text-xs font-bold">
              <i class="fa-solid fa-eye mr-1"></i> View / Download Report
            </a>
          </div>
        `;
        container.appendChild(item);
      });
    }

    function updatePatientDisplayName() {
      const name = document.getElementById('pFullName').value || "Patient";
      document.getElementById('patientProfileDisplay').textContent = name;
      document.getElementById('qrPatientTag').textContent = name;
    }

    function previewPatientPhoto(e) {
      if (e.target.files && e.target.files[0]) {
        const file = e.target.files[0];
        patientAvatarUrl = URL.createObjectURL(file);
        document.getElementById('patientHeaderAvatar').src = patientAvatarUrl;
        document.getElementById('scannedPatientAvatar').src = patientAvatarUrl;
      }
    }

    function generatePatientQR(e) {
      if (e) e.preventDefault();
      
      const payload = {
        n: document.getElementById('pFullName').value,
        a: document.getElementById('pAge').value,
        b: document.getElementById('pBloodGroup').value,
        p: document.getElementById('pPhone').value,
        email: document.getElementById('pEmail').value,
        d: document.getElementById('pDOB').value,
        id: document.getElementById('pPolicyNum').value,
        r: document.getElementById('pVisitReason').value,
        t: new Date().toISOString()
      };

      const qrContainer = document.getElementById('qrcodeBox');
      qrContainer.innerHTML = '';
      
      new QRCode(qrContainer, {
        text: JSON.stringify(payload),
        width: 180,
        height: 180,
        colorDark: "#000000",
        colorLight: "#ffffff",
        correctLevel: QRCode.CorrectLevel.M
      });

      document.getElementById('qrPatientTag').textContent = `${payload.n} (${payload.b}) • Age: ${payload.a}`;
      document.getElementById('qrTimestampTag').textContent = `Pass Generated: ${new Date().toLocaleTimeString()}`;
      updatePatientDisplayName();
    }

    // ================= DOCTORS DIRECTORY & BOOKING =================
    function renderPatientDoctors() {
      const dept = document.getElementById('filterDepartment').value;
      const hosp = document.getElementById('filterHospital').value;
      const list = document.getElementById('patientDoctorsList');
      if (!list) return;
      list.innerHTML = '';

      let filtered = doctors;
      if (dept !== 'All') filtered = filtered.filter(d => d.dept === dept);
      if (hosp !== 'All') filtered = filtered.filter(d => d.hospital === hosp);

      if (filtered.length === 0) {
        list.innerHTML = `
          <div class="col-span-full text-center py-8 text-slate-400 bg-white rounded-2xl border border-slate-200">
            <i class="fa-solid fa-user-doctor text-2xl mb-2"></i>
            <p class="text-xs">No doctors currently listed for the selected department/hospital.</p>
          </div>
        `;
        return;
      }

      filtered.forEach(doc => {
        const isAvailable = doc.status === 'Available';
        const card = document.createElement('div');
        card.className = "bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex flex-col justify-between space-y-3 hover:shadow-md transition";
        card.innerHTML = `
          <div class="flex items-start space-x-3">
            <img src="${doc.photo}" alt="${doc.name}" class="w-14 h-14 rounded-full object-cover border-2 ${isAvailable ? 'border-emerald-500' : 'border-rose-300'}" />
            <div class="flex-1">
              <span class="inline-block text-[10px] font-bold px-2 py-0.5 rounded-full bg-teal-50 text-teal-800 mb-1">
                <i class="fa-solid fa-hospital mr-1"></i> ${doc.hospital}
              </span>
              <h3 class="font-bold text-sm text-slate-900">${doc.name}</h3>
              <p class="text-xs text-teal-700 font-semibold">${doc.dept}</p>
              <p class="text-[11px] text-slate-400">${doc.room}</p>
            </div>
          </div>
          <div class="flex items-center justify-between pt-2 border-t text-xs">
            <span class="font-bold px-2.5 py-0.5 rounded-full ${isAvailable ? 'bg-emerald-100 text-emerald-700' : 'bg-rose-100 text-rose-700'}">
              <i class="fa-solid fa-circle text-[7px] mr-1"></i> ${doc.status}
            </span>
            <button onclick="openBookingModal('${doc.id}')" ${!isAvailable ? 'disabled' : ''} class="px-3.5 py-1.5 rounded-xl font-bold text-xs transition ${isAvailable ? 'bg-teal-700 hover:bg-teal-800 text-white shadow-sm' : 'bg-slate-200 text-slate-400 cursor-not-allowed'}">
              Book Specialist
            </button>
          </div>
        `;
        list.appendChild(card);
      });
    }

    function openBookingModal(docId) {
      currentSelectedDoctor = doctors.find(d => String(d.id) === String(docId));
      const detailBox = document.getElementById('modalDoctorDetails');
      detailBox.innerHTML = `
        <div class="flex items-center space-x-3">
          <img src="${currentSelectedDoctor.photo}" class="w-12 h-12 rounded-full object-cover border-2 border-teal-600" />
          <div>
            <h4 class="font-bold text-slate-900 text-sm">${currentSelectedDoctor.name}</h4>
            <p class="text-xs text-teal-700 font-semibold">${currentSelectedDoctor.dept}</p>
            <p class="text-xs font-bold text-amber-800"><i class="fa-solid fa-hospital mr-1"></i> ${currentSelectedDoctor.hospital}</p>
            <p class="text-[10px] text-slate-400">${currentSelectedDoctor.room}</p>
          </div>
        </div>
      `;
      document.getElementById('bookDate').valueAsDate = new Date();
      document.getElementById('bookingModal').classList.remove('hidden');
    }

    function closeBookingModal() {
      document.getElementById('bookingModal').classList.add('hidden');
    }

    function confirmAppointment() {
      const slot = document.getElementById('bookSlot').value;
      const date = document.getElementById('bookDate').value;
      const pName = document.getElementById('pFullName').value || "Rahul Sharma";
      const pEmail = document.getElementById('pEmail').value || "rahul.sharma@medmail.com";
      const pPhone = document.getElementById('pPhone').value || "+91 98765 43210";
      const pBlood = document.getElementById('pBloodGroup').value || "O+";
      const pAge = document.getElementById('pAge').value || 32;

      const bookingId = 'booking_' + Date.now();
      const newBooking = {
        id: bookingId,
        patientName: pName,
        patientEmail: pEmail,
        phone: pPhone,
        blood: pBlood,
        age: pAge,
        hospital: currentSelectedDoctor.hospital,
        doctorName: currentSelectedDoctor.name,
        dept: currentSelectedDoctor.dept,
        slot: slot,
        date: date,
        treatedStatus: "Not Treated",
        revisitStatus: "Pending Assessment"
      };

      db.ref('bookings/' + bookingId).set(newBooking).then(() => {
        alert(`Appointment confirmed with ${currentSelectedDoctor.name} at ${currentSelectedDoctor.hospital}!`);
        closeBookingModal();
        renderPatientAppointments();
      }).catch(err => {
        alert("Error booking appointment: " + err.message);
      });
    }

    // ================= HOSPITAL MODE =================
    function switchHospitalTab(tabName) {
      document.getElementById('hospPageIngestion').classList.toggle('hidden', tabName !== 'ingestion');
      document.getElementById('hospPageBookings').classList.toggle('hidden', tabName !== 'bookings');
      document.getElementById('hospPageLogs').classList.toggle('hidden', tabName !== 'logs');
      document.getElementById('hospPageSummary').classList.toggle('hidden', tabName !== 'summary');
      document.getElementById('hospPageDoctors').classList.toggle('hidden', tabName !== 'doctors');

      document.getElementById('hospTabIngestion').className = tabName === 'ingestion' ? 'tab-active-hospital px-4 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl' : 'px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl';
      document.getElementById('hospTabBookings').className = tabName === 'bookings' ? 'tab-active-hospital px-4 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl' : 'px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl';
      document.getElementById('hospTabLogs').className = tabName === 'logs' ? 'tab-active-hospital px-4 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl' : 'px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl';
      document.getElementById('hospTabSummary').className = tabName === 'summary' ? 'tab-active-hospital px-4 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl' : 'px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl';
      document.getElementById('hospTabDoctors').className = tabName === 'doctors' ? 'tab-active-hospital px-4 py-2.5 text-xs md:text-sm flex items-center whitespace-nowrap rounded-xl' : 'px-4 py-2.5 text-xs md:text-sm text-slate-500 hover:text-slate-700 flex items-center whitespace-nowrap rounded-xl';

      if (tabName === 'bookings') renderHospitalBookings();
      if (tabName === 'logs') renderHospitalLogs();
      if (tabName === 'summary') syncPrintSummaryData();
      if (tabName === 'doctors') renderHospitalRoster();
    }

    function toggleLiveCameraScanner() {
      const box = document.getElementById('cameraScannerBox');
      const btnLabel = document.getElementById('cameraBtnLabel');

      if (!isCameraRunning) {
        box.classList.remove('hidden');
        btnLabel.textContent = "Stop Camera Scanner";
        html5QrCodeScanner = new Html5Qrcode("qr-reader");
        html5QrCodeScanner.start(
          { facingMode: "environment" },
          { fps: 10, qrbox: { width: 250, height: 250 } },
          (decodedText) => {
            let data;
            try {
              data = JSON.parse(decodedText);
            } catch (err) {
              data = { n: decodedText, r: "Scanned via Live Camera Pass", b: "O+", a: "32", p: "+91 98765 43210" };
            }
            applyScannedPatientData(data);
            if (html5QrCodeScanner) {
              html5QrCodeScanner.stop().then(() => {
                box.classList.add('hidden');
                btnLabel.textContent = "Start Camera Scanner";
                isCameraRunning = false;
              }).catch(e => {
                box.classList.add('hidden');
                btnLabel.textContent = "Start Camera Scanner";
                isCameraRunning = false;
              });
            }
          },
          (error) => {}
        ).catch(err => {
          alert("Camera error: " + err);
          box.classList.add('hidden');
          btnLabel.textContent = "Start Camera Scanner";
          isCameraRunning = false;
        });
        isCameraRunning = true;
      } else {
        if (html5QrCodeScanner) {
          html5QrCodeScanner.stop().then(() => {
            box.classList.add('hidden');
            btnLabel.textContent = "Start Camera Scanner";
            isCameraRunning = false;
          });
        }
      }
    }

    function handleQRImageUpload(event) {
      if (event.target.files && event.target.files[0]) {
        const file = event.target.files[0];
        const html5QrCode = new Html5Qrcode("qr-file-reader");
        html5QrCode.scanFile(file, true)
          .then(decodedText => {
            try {
              const data = JSON.parse(decodedText);
              applyScannedPatientData(data);
              alert(`Successfully scanned QR image for: ${data.n || data.name || 'Patient'}`);
            } catch (e) {
              alert("Decoded content: " + decodedText);
            }
          })
          .catch(err => {
            if (confirm("Could not automatically decode this image. Load active patient profile via Quick Simulation?")) {
              simulateScanPatient();
            }
          });
      }
    }

    function simulateScanPatient() {
      const mockData = {
        n: document.getElementById('pFullName').value,
        a: document.getElementById('pAge').value,
        b: document.getElementById('pBloodGroup').value,
        p: document.getElementById('pPhone').value,
        email: document.getElementById('pEmail').value,
        avatar: patientAvatarUrl,
        d: document.getElementById('pDOB').value,
        id: document.getElementById('pPolicyNum').value,
        r: document.getElementById('pVisitReason').value
      };
      applyScannedPatientData(mockData);
    }

    function applyScannedPatientData(data) {
      const card = document.getElementById('scannedPatientCard');
      card.classList.remove('hidden');
      card.style.display = 'block';

      if (data.avatar) document.getElementById('scannedPatientAvatar').src = data.avatar;

      const pName = data.n || data.name || (typeof data === 'string' ? data : "Rahul Sharma");
      const pAge = data.a || data.age || "32";
      const pBlood = data.b || data.blood || "O+";
      const pPhone = data.p || data.phone || "+91 98765 43210";
      const pEmail = data.email || "rahul.sharma@medmail.com";
      const pReason = data.r || data.visitReason || "Chronic Lumbar Spondylosis with Lower Back Nerve Compression";

      document.getElementById('scanName').textContent = pName;
      document.getElementById('scanMeta').textContent = `Email: ${pEmail} | Age: ${pAge} | Blood: ${pBlood}`;
      document.getElementById('scanComplaints').textContent = pReason;
      document.getElementById('scanBloodAge').textContent = `Blood: ${pBlood} | Age: ${pAge} Yrs | Phone: ${pPhone}`;
      document.getElementById('scanVisitBadge').textContent = `Visit #${patientVisitCount}`;
      document.getElementById('logPurpose').value = pReason;
      document.getElementById('repPatientName').value = pName;

      syncPrintSummaryData();
      setTimeout(() => card.scrollIntoView({ behavior: 'smooth', block: 'center' }), 150);
    }

    function commitVisitLog() {
      const pName = document.getElementById('scanName').textContent;
      const doc = document.getElementById('logDoctorSelect').value;
      const purpose = document.getElementById('logPurpose').value;
      const timeStr = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });

      const logId = 'log_' + Date.now();
      const newLog = {
        id: logId,
        time: timeStr,
        patientName: pName,
        visitNum: patientVisitCount,
        doctor: doc,
        purpose: purpose,
        abha: document.getElementById('pPolicyNum').value
      };

      db.ref('logs/' + logId).set(newLog).then(() => {
        patientVisitCount++;
        document.getElementById('scanVisitBadge').textContent = `Visit #${patientVisitCount}`;
        alert(`Patient check-in recorded to cloud log vault!`);
      });
    }

    function goToHospitalSummaryPrint() {
      syncPrintSummaryData();
      switchHospitalTab('summary');
    }

    function syncPrintSummaryData() {
      document.getElementById('printHospitalTitle').textContent = currentHospitalName.toUpperCase();
      document.getElementById('printHospitalLogo').src = hospitalLogoUrl;
      document.getElementById('printCurrentDate').textContent = new Date().toLocaleDateString();
      
      document.getElementById('printName').textContent = document.getElementById('pFullName').value || "Rahul Sharma";
      document.getElementById('printBloodGroup').textContent = document.getElementById('pBloodGroup').value || "O+";
      document.getElementById('printAge').textContent = `${document.getElementById('pAge').value || '32'} Years`;
      document.getElementById('printPhone').textContent = document.getElementById('pPhone').value || "+91 98765 43210";
      
      const doctorSelect = document.getElementById('logDoctorSelect');
      const doctorTreated = doctorSelect && doctorSelect.value ? doctorSelect.value : (doctors[0] ? doctors[0].name + " (" + doctors[0].hospital + ")" : "Dr. Vikram Sethi");
      document.getElementById('printDoctorTreated').textContent = doctorTreated;
      
      const diseaseInput = document.getElementById('logPurpose').value || document.getElementById('pVisitReason').value;
      document.getElementById('printDisease').textContent = diseaseInput;
    }

    function renderHospitalBookings(bookingsToRender = null) {
      const tbody = document.getElementById('hospitalBookingsBody');
      const badge = document.getElementById('hospitalBookingsBadge');
      if (!tbody) return;
      tbody.innerHTML = '';

      const hospitalList = patientBookings.filter(b => b.hospital.toLowerCase() === currentHospitalName.toLowerCase());
      const list = bookingsToRender || hospitalList;
      badge.textContent = hospitalList.length;

      if (list.length === 0) {
        tbody.innerHTML = `<tr><td colspan="7" class="p-6 text-center text-slate-400">No appointments booked for ${currentHospitalName} yet.</td></tr>`;
        return;
      }

      list.forEach(b => {
        const isTreated = b.treatedStatus === "Treated";
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td class="p-3">
            <span class="font-bold text-slate-900 block">${b.date}</span>
            <span class="text-[10px] text-slate-400 font-mono">${b.slot}</span>
          </td>
          <td class="p-3">
            <p class="font-bold text-slate-900">${b.patientName}</p>
            <p class="text-[10px] text-slate-500">Age: ${b.age} | Blood: <strong class="text-rose-600">${b.blood}</strong> | ${b.phone}</p>
          </td>
          <td class="p-3 font-semibold text-teal-900">${b.doctorName}</td>
          <td class="p-3 text-slate-600">${b.dept}</td>
          <td class="p-3 text-center">
            <button onclick="toggleTreatedStatus('${b.id}')" class="px-2.5 py-1 rounded-full text-[10px] font-extrabold transition ${isTreated ? 'bg-emerald-100 text-emerald-800 hover:bg-emerald-200' : 'bg-rose-100 text-rose-800 hover:bg-rose-200'}">
              <i class="fa-solid ${isTreated ? 'fa-circle-check' : 'fa-clock'} mr-1"></i> ${b.treatedStatus}
            </button>
          </td>
          <td class="p-3 text-center">
            <button onclick="toggleRevisitStatus('${b.id}')" class="px-2.5 py-1 rounded-xl text-[10px] font-bold border transition ${b.revisitStatus.includes('Revisitable') ? 'bg-amber-50 text-amber-900 border-amber-300' : 'bg-slate-50 text-slate-600 border-slate-200'}">
              <i class="fa-solid fa-arrows-rotate mr-1"></i> ${b.revisitStatus}
            </button>
          </td>
          <td class="p-3 text-center space-x-1">
            <button onclick="openQuickUpload('${b.patientName}', '${b.patientEmail || ''}')" title="Attach Lab/Scan Report" class="bg-blue-50 hover:bg-blue-100 text-blue-700 px-2.5 py-1 rounded-lg text-[10px] font-bold transition">
              <i class="fa-solid fa-paperclip mr-1"></i> Attach Report
            </button>
          </td>
        `;
        tbody.appendChild(tr);
      });
    }

    function toggleTreatedStatus(bookingId) {
      const item = patientBookings.find(b => String(b.id) === String(bookingId));
      if (item) {
        item.treatedStatus = item.treatedStatus === "Treated" ? "Not Treated" : "Treated";
        db.ref('bookings/' + bookingId).update({ treatedStatus: item.treatedStatus });
      }
    }

    function toggleRevisitStatus(bookingId) {
      const item = patientBookings.find(b => String(b.id) === String(bookingId));
      if (item) {
        item.revisitStatus = item.revisitStatus.includes("Revisitable") ? "No Follow-up Required" : "Revisitable (Follow-up 7 Days)";
        db.ref('bookings/' + bookingId).update({ revisitStatus: item.revisitStatus });
      }
    }

    function filterHospitalBookings() {
      const query = document.getElementById('bookingSearchInput').value.toLowerCase();
      const hospitalList = patientBookings.filter(b => b.hospital.toLowerCase() === currentHospitalName.toLowerCase());
      const filtered = hospitalList.filter(b => 
        b.patientName.toLowerCase().includes(query) || b.doctorName.toLowerCase().includes(query) || b.dept.toLowerCase().includes(query)
      );
      renderHospitalBookings(filtered);
    }

    function renderHospitalLogs(logsToRender = visitLogs) {
      const tbody = document.getElementById('hospitalLogBody');
      const badge = document.getElementById('totalLogsBadge');
      if (!tbody) return;
      badge.textContent = visitLogs.length;
      tbody.innerHTML = '';

      if (logsToRender.length === 0) {
        tbody.innerHTML = `<tr><td colspan="6" class="p-4 text-center text-slate-400">No logs found.</td></tr>`;
        return;
      }

      logsToRender.forEach(log => {
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td class="p-3 font-mono text-slate-500 text-[11px]">${log.time}</td>
          <td class="p-3">
            <p class="font-bold text-slate-900">${log.patientName}</p>
            <p class="text-[10px] text-slate-400 font-mono">${log.abha}</p>
          </td>
          <td class="p-3"><span class="bg-amber-100 text-amber-900 px-2 py-0.5 rounded font-bold text-[10px]">Visit #${log.visitNum}</span></td>
          <td class="p-3 text-teal-800 font-semibold">${log.doctor}</td>
          <td class="p-3 text-slate-600">${log.purpose}</td>
          <td class="p-3 text-center space-x-1">
            <button onclick="openQuickUpload('${log.patientName}')" class="bg-blue-50 hover:bg-blue-100 text-blue-700 px-2.5 py-1 rounded-lg text-[10px] font-bold">
              <i class="fa-solid fa-file-arrow-up mr-1"></i> Attach Report
            </button>
          </td>
        `;
        tbody.appendChild(tr);
      });
    }

    function filterHospitalLogs() {
      const query = document.getElementById('logSearchInput').value.toLowerCase();
      const filtered = visitLogs.filter(l => l.patientName.toLowerCase().includes(query) || l.doctor.toLowerCase().includes(query));
      renderHospitalLogs(filtered);
    }

    function exportLogsCSV() {
      let csv = "Time,Patient Name,Visit Number,Assigned Doctor,Disease Diagnosis,ABHA ID\n";
      visitLogs.forEach(l => {
        csv += `"${l.time}","${l.patientName}","${l.visitNum}","${l.doctor}","${l.purpose}","${l.abha}"\n`;
      });
      const blob = new Blob([csv], { type: 'text/csv' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `OPD_Logs_${new Date().toISOString().split('T')[0]}.csv`;
      a.click();
    }

    let quickUploadTargetEmail = "";
    function openQuickUpload(patientName, patientEmail = "") {
      quickUploadTargetPatient = patientName;
      quickUploadTargetEmail = patientEmail;
      document.getElementById('quickUploadPatientName').textContent = patientName;
      document.getElementById('quickUploadModal').classList.remove('hidden');
    }

    function closeQuickUploadModal() {
      document.getElementById('quickUploadModal').classList.add('hidden');
    }

    function saveQuickReport() {
      const cat = document.getElementById('quickRepCategory').value;
      const fileInp = document.getElementById('quickFileInput');
      let fileUrl = "https://images.unsplash.com/photo-1516549655169-df83a0774514?w=600&auto=format&fit=crop&q=80";
      
      if (fileInp.files && fileInp.files[0]) {
        fileUrl = URL.createObjectURL(fileInp.files[0]);
      }

      const repId = 'rep_' + Date.now();
      const newReport = {
        id: repId,
        patientName: quickUploadTargetPatient,
        patientEmail: quickUploadTargetEmail || (currentLoggedUser ? currentLoggedUser.email : ""),
        title: `${cat} - Attached Record`,
        category: cat,
        doctor: "Hospital OPD Diagnostic Team",
        date: new Date().toISOString().split('T')[0],
        fileUrl: fileUrl,
        type: "document"
      };

      db.ref('reports/' + repId).set(newReport).then(() => {
        alert(`Report uploaded directly to ${quickUploadTargetPatient}'s isolated patient vault!`);
        closeQuickUploadModal();
      });
    }

    // ================= DOCTORS MANAGEMENT =================
    function renderHospitalRoster() {
      const list = document.getElementById('hospitalDoctorRosterList');
      const countBadge = document.getElementById('rosterCountBadge');
      if (!list) return;
      list.innerHTML = '';
      
      const hospitalDocs = doctors.filter(d => d.hospital.toLowerCase() === currentHospitalName.toLowerCase());
      countBadge.textContent = hospitalDocs.length;

      if (hospitalDocs.length === 0) {
        list.innerHTML = `<p class="text-xs text-slate-400 col-span-2">No doctors currently registered under ${currentHospitalName}.</p>`;
        return;
      }

      hospitalDocs.forEach(d => {
        const row = document.createElement('div');
        row.className = "flex items-center justify-between p-3 bg-slate-50 rounded-2xl border border-slate-100";
        row.innerHTML = `
          <div class="flex items-center space-x-2.5">
            <img src="${d.photo}" class="w-10 h-10 rounded-full object-cover border" />
            <div>
              <p class="font-bold text-slate-800">${d.name}</p>
              <p class="text-[10px] text-teal-700 font-semibold">${d.dept}</p>
              <p class="text-[10px] text-slate-400">${d.room}</p>
            </div>
          </div>
          <div class="flex items-center space-x-1.5">
            <button onclick="toggleDoctorStatus('${d.id}')" class="text-[10px] px-2.5 py-1 rounded-full font-bold transition ${d.status === 'Available' ? 'bg-emerald-100 text-emerald-800 hover:bg-emerald-200' : 'bg-rose-100 text-rose-800 hover:bg-rose-200'}">
              ${d.status}
            </button>
            <button onclick="deleteDoctor('${d.id}')" title="Delete Doctor Profile" class="text-rose-600 hover:bg-rose-100 p-1.5 rounded-lg text-xs transition">
              <i class="fa-solid fa-trash-can"></i>
            </button>
          </div>
        `;
        list.appendChild(row);
      });
    }

    function toggleDoctorStatus(id) {
      const doc = doctors.find(d => String(d.id) === String(id));
      if (doc) {
        doc.status = doc.status === 'Available' ? 'On Leave' : 'Available';
        db.ref('doctors/' + id).update({ status: doc.status });
      }
    }

    function deleteDoctor(id) {
      const target = doctors.find(d => String(d.id) === String(id));
      if (!target) return;
      if (confirm(`Remove ${target.name} from roster?`)) {
        db.ref('doctors/' + id).remove();
      }
    }

    function populateDoctorSelect() {
      const select = document.getElementById('logDoctorSelect');
      if (!select) return;
      select.innerHTML = '';
      doctors.filter(d => d.status === 'Available' && d.hospital.toLowerCase() === currentHospitalName.toLowerCase()).forEach(d => {
        const opt = document.createElement('option');
        opt.value = `${d.name} (${d.dept} - ${d.hospital})`;
        opt.textContent = `${d.name} (${d.dept})`;
        select.appendChild(opt);
      });
    }

    function addNewDoctor(e) {
      e.preventDefault();
      const name = document.getElementById('docName').value;
      const hospital = document.getElementById('docHospitalName').value || currentHospitalName;
      const dept = document.getElementById('docDept').value;
      const status = document.getElementById('docStatus').value;
      const room = document.getElementById('docRoom').value || "OPD Chamber";
      const fileInput = document.getElementById('docPhotoInput');

      let photoUrl = "https://images.unsplash.com/photo-1622253692010-333f2da6031d?w=150&auto=format&fit=crop&q=80";
      if (fileInput.files && fileInput.files[0]) {
        photoUrl = URL.createObjectURL(fileInput.files[0]);
      }

      const docId = 'doc_' + Date.now();
      const newDoc = { id: docId, name, hospital, dept, status, room, photo: photoUrl };

      db.ref('doctors/' + docId).set(newDoc).then(() => {
        alert(`Doctor ${name} published to Patient Directory!`);
        document.getElementById('docName').value = '';
        document.getElementById('docRoom').value = '';
        fileInput.value = '';
      });
    }

    function previewUploadFile(e) {
      if (e.target.files && e.target.files[0]) {
        const file = e.target.files[0];
        uploadedFileBlobUrl = URL.createObjectURL(file);
        const note = document.getElementById('filePreviewNote');
        note.textContent = `Ready: ${file.name} (${(file.size / 1024).toFixed(1)} KB)`;
        note.classList.remove('hidden');
      }
    }

    function handleHospitalReportUpload(e) {
      e.preventDefault();
      const patientName = document.getElementById('repPatientName').value;
      const category = document.getElementById('repCategory').value;
      const doctor = document.getElementById('repDoctor').value;
      const today = new Date().toISOString().split('T')[0];

      const repId = 'rep_' + Date.now();
      const newReport = {
        id: repId,
        patientName: patientName,
        patientEmail: currentLoggedUser ? currentLoggedUser.email : "",
        title: `${category} - Diagnostic Record`,
        category: category,
        doctor: doctor,
        date: today,
        fileUrl: uploadedFileBlobUrl || "https://images.unsplash.com/photo-1516549655169-df83a0774514?w=600&auto=format&fit=crop&q=80",
        type: "document"
      };

      db.ref('reports/' + repId).set(newReport).then(() => {
        alert(`Diagnostic report uploaded directly to ${patientName}'s vault!`);
        e.target.reset();
        document.getElementById('filePreviewNote').classList.add('hidden');
      });
    }

    function downloadSelf() {
      const htmlContent = document.documentElement.outerHTML;
      const blob = new Blob([htmlContent], { type: 'text/html' });
      const a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = 'ayushcare_opd_final_portal_v9.html';
      a.click();
    }

    window.addEventListener('DOMContentLoaded', () => {
      renderPatientDoctors();
      renderPatientReports();
      renderHospitalBookings();
      renderHospitalRoster();
      renderHospitalLogs();
      populateDoctorSelect();
      updatePatientDisplayName();
      generatePatientQR();
    });
  </script>
</body>
</html>
