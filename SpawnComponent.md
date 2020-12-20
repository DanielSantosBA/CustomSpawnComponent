# CustomSpawnComponent
#CompanyBR

import bge
from collections import OrderedDict
from random import randint


#SCENE
if not hasattr(bge, "__component__"):
    global objI
    global objA
    
    objI = bge.logic.getCurrentScene().objectsInactive
    objA = bge.logic.getCurrentScene().objects
    


class Spawn(bge.types.KX_PythonComponent):
    args = OrderedDict([
    ("ON/OFF",    False),
    ("Tap",       False),
    ("ActName",      ''),
    ("Behavior", {'Seek', 'Flee', 'PathFollowing'}),
    ("Target",       ''),
    ("Navmesh",      ''),
    ("SpawnTimer", 10.0),
    ("lifeTimer", 100.0),
    ("Align", {'X', 'Y'}),
    ("Invert", False)
    
    ])

    def start(self, args):               

        #ON / OFF
        self.tap     = args['Tap']
        self.onOff   = args["ON/OFF"]
        self.list    = []        

        # TIMER
        self.timer  = args['SpawnTimer']
        self.ltimer = args['lifeTimer']
        
        self.object['timer'] = 0.0
        
       
        # FUNCTIONS OBJECT ACTUATOR 
        self.behavior= args['Behavior']
        self.target  = args['Target']
        self.navmesh = args['Navmesh']
        self.actName = args['ActName']
        
        # ALIGN AXIS TO VECT
        self.align = args['Align']
        self.invert = args['Invert']
        
                     
        for obj in objI:              
            if 'spawn' in obj:
                self.list.append(str(obj))
        self.add = None


    def update(self):
                    
        if self.onOff == True:
            
            # TIMER     
            self.object['timer'] += 0.01

            
            # ACTIVE ADD ACTUATOR
            scene = bge.logic.getCurrentScene()
            
            if self.object['timer'] > self.timer:                               
                
                # DEF OBJECT ADD
                lenList = len(self.list)-1 
                select = self.list[randint(0, lenList)]                                
                
                # LAST OBJECT CREATED
                self.add = scene.addObject(select, self.object)
                self.object['timer'] = 0.0
                
                
                
                
                # STEERING
                if not self.add == None:
                    for child in self.add.children:
                        if 'Steering' in child.name:
                            self.steer = child
                        
                        
                
                ################## LAST OBJECT DEFINITIONS #################
                    self.act = self.steer.actuators[self.actName]                
                    self.act.target =  objA[self.target]
                    
                    
                    # STEERING BEHAVIOR
                    if self.behavior == 'Seek':
                        self.act.behavior = 1

                    if self.behavior == 'Flee':
                        self.act.behavior = 2

                    if self.behavior == 'PathFollowing':
                        self.act.navmesh = objA[self.navmesh]
                        self.act.behavior = 3           
 
                    
            if not self.add == None:
                self.add.components['Physic'].ltimer = self.ltimer
                
                
                if self.object['timer'] >= 0.01 and self.object['timer'] <= 0.02:
                    
                    
                    if self.align == 'X':
                        if self.invert == False:
                            self.add.alignAxisToVect([1,0,0], 1, 1)
                        else:
                            self.add.alignAxisToVect([-1,0,0], 1, 1)

                    if self.align == 'Y':
                        if self.invert == False:
                            self.add.alignAxisToVect([0,1,0], 1, 1)
                        else:
                            self.add.alignAxisToVect([0,-1,0], 1, 1)
                    
                    if self.tap == True:
                        self.onOff = False
                
                self.add.alignAxisToVect([0,0,1], 2, 1)
